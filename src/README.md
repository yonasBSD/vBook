# What is vSMTP ?

vSMTP is a next-gen Mail Transfer Agent (MTA) developed by viridIT teams. You
can follow us on [viridit.com](https://www.viridit.com).

## Why develop a new MTA ?

Whereas optimizing allocated resources is becoming a growing challenge, computer
attacks remain a constant issue. Over 300 billion emails are sent and received
in the world every day. Billions of attachments are processed, analyzed and
delivered, contributing to the increase in greenhouse gas emissions. To meet
this challenge, viridIT is developing a new technology of email gateways, also
called vSMTP.

## Why vSMTP is your future SMTP server ?

Because it is secured, faster and greener.

- It is developed in Rust, implying high performance and stability.
- It is modular and highly customizable.
- It includes a complete filtering system.
- It is actively maintained and developed.

## Documentation

About the code and related issues, please check the
[project Wiki](https://github.com/viridIT/vSMTP/wiki) and use the GitHub issue
tracker. To stay tuned, ask questions and get in-depth answers feel free to
register and visit our
[community forums](https://www.viridit.com/community-forum). You can also open a
GitHub [discussion](https://github.com/viridIT/vSMTP/discussions).\
For documentation, user guide, etc. please consult
[GitHub wiki](https://github.com/viridIT/vSMTP/wiki).

## Commercial

For any question related to commercial, licensing, etc. you can join us at
<https://www.viridit.com/contact>.

## Roadmap

vSMTP is currently under development. The current versions "0.7.x" focus on the
SMTP connection and state machine. You can find more information about the
project agenda in the
[ROADMAP](https://github.com/viridIT/vSMTP/blob/main/ROADMAP.md).

## License

The standard version of vSMTP is free and under an Open Source license.

It is provided as usual without any warranty. Please refer to the
[LICENSE](https://github.com/viridIT/vSMTP/blob/main/LICENSE) file for further
information.


Be as secure as possible. Code carefully, do strict validity checks especially in the network input path, and use bounded buffer operations. Use privilege separation to mitigate the effects of possible security bugs.
Reliability is extremely important. Any email that OpenSMTPD has accepted has to be handled with care and must not be lost.
Provide a lean implementation, sufficient for a majority. Don't try to support each and every obscure usage case, but cover the typical ones.
Provide a powerful and easy to understand configuration language.
Be fast and efficient. OpenSMTPD must be able to handle large queues with reasonable performance.

OpenSMTPD is developed with the same rigorous security process that the OpenBSD group is famous for. If you wish to report a security issue in OpenSMTPD, please contact the private developers list <opensmtpd-security@openbsd.org>.