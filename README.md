# ByeLog4Shell
Use Log4Shell vulnerability to vaccinate a victim server against Log4Shell

## Description 

A vulnerability impacting Apache Log4j versions 2.0 through 2.14.1 was disclosed on the project’s Github on December 9, 2021. 
The flaw has been dubbed “Log4Shell,”, and has the highest possible severity rating of 10. Software made or
managed by the Apache Software Foundation (From here on just "Apache") is pervasive and comprises nearly a third of all
web servers in the world—making this a potentially catastrophic flaw.
The Log4Shell vulnerability CVE-2021-44228 was published on 12/9/2021 and allows remote code execution on vulnerable servers.


While the best mitigation against these vulnerabilities is to patch log4j to
2.17.0 and above, in Log4j version (>=2.10) this behavior can be partially mitigated by
setting system property `log4j2.formatMsgNoLookups` to `true` or by removing
the JndiLookup class from the classpath. 

On 12/14/2021 the Apache software foundation disclosed CVE-2021-45046 which was patched in log4j version 2.16.0. This
vulnerability showed that in certain scenarios, for example, where attackers can control a thread-context variable that
gets logged, even the flag `log4j2.formatMsgNoLookups` is insufficient to mitigate log4shell. An
additional CVE, less severe, CVE-2021-45105 was discovered. This vulnerability exposes the server to
an infinite recursion that could crash the server is some scenarios. It is recommened to upgrade to
2.17.0
However, enabling these system property requires access to the vulnerable servers as well as a restart. 
I developed the following code that _exploits_ the same vulnerability and the payload therein
sets the vulnerable setting as disabled. The payload then searches
for all `LoggerContext` and removes the JNDI `Interpolator` preventing even recursive abuses. 
this effectively blocks any further attempt to exploit Log4Shell on the server. 

However, this project attempts to fix the vulnerability by using the bug against itself. Cool?

## Supported versions
ByeLog4Shell supports log4j version 2.0 - 2.14.1

## How it works
On versions (>= 2.10.0) of log4j that support the configuration `FORMAT_MESSAGES_PATTERN_DISABLE_LOOKUPS`, this value is
set to `True` disabling the lookup mechanism entirely. As disclosed in CVE-2021-45046, setting this flag is insufficient,
therefore the payload searches all existing `LoggerContexts` and removes the JNDI key from the `Interpolator` used to
process `${}` fields. This means that even other recursive uses of the JNDI mechanisms will fail.
Then, the log4j jarfile will be remade and patched. The patch is included in this
git repository, however it is not needed in the final build because the real patch
is included in the payload as Base64.

In persistence mode (see [below](#transient-vs-persistent-mode)), the payload additionally attempts to locate the `log4j-core.jar`,
remove the `JndILookup` class, and modify the PluginCache to completely remove the JNDI plugin. Upon subsequent JVM
restarts the `JndiLookup` class cannot be found and log4j will not support for JNDI

## Transient vs Persistent mode
This package generates two flavors of the payload - Transient and Persistent. 
In Transient mode, the payload modifies
the current running JVM. The payload is very delicate to just touch the logger context and configuration. We thus
believe the risk of using the Transient mode are very low on production environments.

Persistent mode performs all the changes of the Transient mode and *in addition* searches for the jar from which `log4j`
loads the `JndiLookup` class. It then attempts to modify this jar by removing the `JndiLookup` class as well as
modifying the plugin registry. There is inherently more risk in this approach as if the `log4j-core.jar` becomes
corrupted, the JVM may crash on start.

The choice of which mode to use is selected by the flag you Pass on The Script

## How to use

1. Download this repository and Run the ByeLog4Shell Script with the --install Flag

2. `git clone https://github.com/cybereason/ByeLog4Shell.git`

3. `cd ByeLog4Shell`
  
4. `bash ByeLog4Shell --install`

5. Run `bash ByeLog4Shell --help` and "Play" around and close the Vulnerability!

Happy Hacking for the Good


## DISCLAIMER: 
The code described in this advisory (the “Code”) is provided on an “as is” and
“as available” basis may contain bugs, errors and other defects. You are
advised to safeguard important data and to use caution. By using this Code, you
agree that Qerim Iseni09 shall have no liability to you for any claims in
connection with the Code. Cybereason disclaims any liability for any direct,
indirect, incidental, punitive, exemplary, special or consequential damages,
even if Qerim Iseni09 or its related parties are advised of the possibility of
such damages. Qerim Iseni09 undertakes no duty to update the Code or this
advisory.

## License
The source code for the site is licensed under the MIT license, which you can find in the LICENSE file.

