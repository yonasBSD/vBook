# vSL: the vSMTP Scripting Language

vSL is a lightweight scripting language dedicated to email filtering. It is based on the [RHAI] scripting language. vSL combines declarative rules with objects and actions.

[RHAI]: (https://rhai.rs/)

vSL has no notion of a "main" program. vSL files are analyzed and executed by vSMTP through specific function calls. However, advanced users can use the [RHAI] scripting language on top of vSL to create and manage a wide variety of actions.