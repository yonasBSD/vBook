# Rules

Rules are the entry point to interact with the SMTP traffic at a user level.
Nevertheless specific parameters like timeout, system logging, tls configuration, etc. are set in the configuration files.

## Overall Syntax

Rules follow a specific syntax :

```vsl
rule <stage> <name> #{
    condition: || <condition>,
    on_success: || <action>,
    on_failure: || <action>,
};
```

>The symbols || after "condition", "on_success" and "on_failure" keywords must be understood as closure delimiters and not as boolean operators.

## vSL stages vs SMTP states

The end user can analyze and interact with SMTP traffic at several stages. To do this, vSL publishes a global context which is updated with each step change.

| Stage | SMTP state | Context available
| :--- | :--- | :---
| connect | Before HELO/EHLO command | Connection related information.
| helo | After HELO/EHLO command | HELO string.
| mail | After MAIL FROM command | Sender address.
| rcpt | After RCPT TO command | The entire SMTP envelop.
| preq | Before queuing the mail (1) | The entire mail.
| postq | After queuing the mail (2) | The entire mail. Not implemented in version 0.7.x.

> (1) After end of data, before server answer (ex. 250 OK).
> (2) Connection is already closed and the SMTP code sent.

## Conditions

The "condition: ||" primitive expects a boolean after the || symbol.
Booleans can come directly from [RHAI](https://rhai.rs/) or vSL functions as shown hereunder (like vsl.IS_CONNECT).

### Built-in vSL conditions

Foreach stage a vSL condition that match the message context is available.
The function syntax is : IS_*STAGE*(object).

```vsl
obj ip4 "localhost" "192.168.1.34";

rule connect "check on connect" #{
    condition:  || vsl.IS_CONNECT("localhost"),
    on_success: || vsl.ACCEPT(),
    on_failure: || vsl.DENY()
};
```

As explained previously, the values obtained from the previous steps are still available in the current stage.

```vsl
obj ip4 "localhost" "192.168.1.34";
obj addr "foo" "foo@bar.com";

rule mail "adv check" #{
    condition:  || vsl.IS_CONNECT("localhost") && vsl.IS_MAIL("foo"),
    on_success: || vsl.ACCEPT(),
    on_failure: || vsl.DENY()
};
```

### Complex conditions using [RHAI](https://rhai.rs/) functions

```vsl
obj fqdn "foobar" "my.foo.bar";

fn my_function(x) {
    if (x == "foo") { true } else { false }
}

[...]
rule mail "adv check" #{
    condition: || !vsl.IS_HELO("foobar") && my_function("bar"),
[...]
```

> Please note that && and || operators are short-circuits.
> In this case my_function() function will not be evaluated if the 1st part already proves the condition wrong.
> To counter this behavior use the boolean operators & and |.

## Rule actions : on_success and on_failure

These methods interact with the SMTP engine.
Those methods must return a state (see the Rule Engine Actions section) to influence vSMTP.

for example:

```vsl
obj ip4 "localhost" "192.168.1.34";

fn my_action() {
    vsl.DUMP(`/tmp/mail/dump/${msg_id}`);
    vsl.FACCEPT()
}

rule connect "check on connect" #{
    condition:  || vsl.IS_CONNECT("localhost"),
    on_success: || my_action(),
    on_failure: || {
        vsl.LOG(`Connection from this host is not allowed.`, "stdout");
        vsl.DENY()
    },
};
```

the connection is accepted if it is local, and denied otherwise.
> Note the absence of the semicolon after DENY() since the rule must return a state.

## Implicit rule in a stage

To avoid undefined behavior, the implicit action in a stage is CONTINUE(). If there's no state (i.e.  ACCEPT, DENY, etc.) returned in a stage, the default behavior is to proceed to the next stage, and in the end the message is delivered.

## Context variables

Message related variables are available depending on the stage.

### CONNECT stage

| Name | Type | Description
| :--- | :--- | :---
| ${connect} | ip4/ip6 | Source IP address.
| ${port} | int | Source port.
| ${date} | string | Current date.
| ${time} | string | Current time.

### HELO stage

| Name | Type | Description
| :--- | :--- | :---
| ${helo} | string | HELO/EHLO SMTP value |

### MAIL stage

| Name | Type | Description
| :--- | :--- | :---
| ${mail.full}| addr | Sender email address. ${mail} is equivalent.
| ${mail.local_part} | string | Sender name.
| ${mail.domain} | fqdn | Sender fqdn.

### RCPT stage

| Name | Type | Description
| :--- | :--- | :---
| ${rcpt.full} | addr | Current recipient address. ${rcpt} is equivalent.
| ${rcpt.local_part} | string | Current sender name.
| ${rcpt.domain} | fqdn | Current sender fqdn.
| ${rcpts.full} | Array of addr| Recipients addresses. ${rcpts} is equivalent.
| ${rcpts.local_parts} | Array of string | Senders name.
| ${rcpts.domains} | Array of fqdn | Senders fqdn.

>Please note that the `rcpts` array is completely filled at PREQ stage and not in RCPT stage.

### PREQ stage

| Name | Type | Description
| :--- | :--- | :---
| preq | ${data} | Email body.

in this stage, every built-in variables are filled. You can do complex rule matching in this stage.

## About DUMP action

The DUMP action writes the content of an email in JSON format with the content available at the specified rule stage.

```vsl
rule mail "dump_at_mail_stage" #{
    condition: true,
    on_success: vsl.DUMP(`/var/spool/mta/`),
};
```

This rule will dump in JSON format only the content available at the "mail" stage : `connect`, `helo` and `mail` parameters.

## About BLOCK action

The BLOCK action pushes the mail in a user defined quarantine directory.
Unlike DUMP:

- The ENTIRE content of the email is written in JSON format regardless of the stage declared in the rule (including the envelop and body).
- All rules are skipped, and the server delivers a 554 smtp code to the client.

```vsl
rule helo "my_quarantine" #{
    condition: vsl.IS_HELO("blacklist"),
    on_success: || vsl.BLOCK("/var/spool/mta/blacklist/"),
    ...
};
```