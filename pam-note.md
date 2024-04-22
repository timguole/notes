# PAM Note

## Classical control keywords

| Keyword    | succeed                                                      | fail                                                        |
| ---------- | ------------------------------------------------------------ | ----------------------------------------------------------- |
| required   | continue                                                     | continue                                                    |
| requisite  | continue                                                     | return to app or superior stack                             |
| sufficient | return (depends on previous required module result)          | ignored and continue                                        |
| optional   | if there is only one module, return success. Otherwise, ignore | if there is only one module, return fail. Otherwise, ignore |
| include    | N/A. it includes rules of given type from a file             | N/A. it includes rules of given type from a file            |
| substack   | N/A. die/done in substack does not affect the outer stack    | N/A. die/done in substack does not affect the outer stack   |

## Complex syntax

```shell
[value1=action1 value2=action2 ...]
```

Values is the return code from the function called in the module. Vaules can be one of the following:

```shell
 success, open_err, symbol_err, service_err, system_err, buf_err, perm_denied, auth_err, cred_insufficient, authinfo_unavail, user_unknown, maxtries, new_authtok_reqd, acct_expired, session_err, cred_unavail, cred_expired, cred_err, no_module_data, conv_err, authtok_err, authtok_recover_err, authtok_lock_busy, authtok_disable_aging, try_again, ignore, abort, authtok_expired, module_unknown, bad_item, conv_again, incomplete, and default.
```

The 'default' means all other values not mentioned explicitly in the [].

Actions can be one of the following:

- bad: fail and continue
- die: fail and terminate the stack processing
- ignore:
- ok: if former value of the stack means success, override the former value with this value. Otherwise, still fail
- done: like ok, but terminate the stack processing
- N: a number, means jump over the next N lines of modules
- reset: clear all state of the current stack and start again with the next stacked module