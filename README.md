# Ansible Role: Installs Apache Tomcat Java Application server (optionally with Hugepages)

Installs Apache Tomcat Java Application server. Most complete Tomcat installation, supporting, init.d script, application naming, hugepages, hardening, beautiful error pages, sha512 hashed passwords, JMX configuration, multiple Tomcat versions, separated catalina_home and caralina_base.

[![Build Status](https://travis-ci.org/KAMI911/ansible-role-tomcat.svg?branch=master)](https://travis-ci.org/KAMI911/ansible-role-tomcat)

## Table of Contents

1. [Requirements][Requirements]
2. [Installation][Installation]
3. [Role Variables][Role Variables]
4. [Dependencies][Dependencies]
5. [Example Playbook][Example Playbook]
6. [Licensing][Licensing]
7. [Author Information][Author Information]
8. [Support][Support]
9. [Contributing][Contributing]
10. [Donation][Donation]

## Requirements

None.

## Installation

    ansible-galaxy install kami911.tomcat

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    tomcat_majorversion: 8

Tomcat major version.

    tomcat_minorversion: 5

Tomcat minor version.

    tomcat_patchversion: 4

Tomcat micro version.

    tomcat_use_huge_pages: True

Use Huge Pages (Java calls it: UseLargePages) for enhance performance of Java applications. When a process uses some memory, the CPU is marking the RAM as used by that process. For efficiency, the CPU allocate RAM by chunks of 4K bytes (it's the default value on many platforms). Those chunks are named pages. Those pages can be swapped to disk, etc.

Since the process address space are virtual, the CPU and the operating system have to remember which page belong to which process, and where it is stored. Obviously, the more pages you have, the more time it takes to find where the memory is mapped. When a process uses 1GB of memory, that's 262144 entries to look up (1GB / 4K). If one Page Table Entry consume 8bytes, that's 2MB (262144 * 8) to look-up.

[Debian Wiki: Hugepages](https://wiki.debian.org/Hugepages)

When you enable it use [KAMI911:hugepages](https://galaxy.ansible.com/KAMI911/hugepages/) to configure Huge Pages in Linux.

    tomcat_http_port: 8080

Tomcat listening http connection port.

    tomcat_http_stp: false

Tomcat "Connector" using the shared thread pool for http connections.

    tomcat_http_zone: "internal"

Tomcat listening http connection port firewall zone.

    tomcat_http_source:  # Tweak this according yout network
    - "0.0.0.0/0"

Tomcat listening http connection port firewall source networks.

    tomcat_http_connection_timeout: 20000

Tomcat listening http connection port connection timeout.

    tomcat_http_compression_mime_types: "text/html,text/xml,text/css,text/javascript,application/x-javascript,application/javascript"

Tomcat http connection compressed MIME-types.

    tomcat_http_compression: true

Tomcat http connection compressed content enable/disable.

    tomcat_http_compression_min_size: 256

Tomcat http connection min compressed file size.

    tomcat_http_compression_nocompress_user_agent: ""

Tomcat http connection must be uncompressed for these browser user agents.

    tomcat_file_encoding: UTF-8

Tomcat file encoding parameter: UTF-8

    tomcat_page_encoding: UTF-8

Tomcat page encoding parameter: UTF-8

    tomcat_access_log_pattern: "%h %l %u %t &quot;%r&quot; %s %b"

Pattern string of Tomcat access log. Tomcat [Access Logging](https://tomcat.apache.org/tomcat-9.0-doc/config/valve.html#Access_Logging):

Values for the pattern attribute are made up of literal text strings, combined with pattern identifiers prefixed by the "%" character to cause replacement by the corresponding variable value from the current request and response. The following pattern codes are supported:

    %a - Remote IP address
    %A - Local IP address
    %b - Bytes sent, excluding HTTP headers, or '-' if zero
    %B - Bytes sent, excluding HTTP headers
    %h - Remote host name (or IP address if enableLookups for the connector is false)
    %H - Request protocol
    %l - Remote logical username from identd (always returns '-')
    %m - Request method (GET, POST, etc.)
    %p - Local port on which this request was received. See also %{xxx}p below.
    %q - Query string (prepended with a '?' if it exists)
    %r - First line of the request (method and request URI)
    %s - HTTP status code of the response
    %S - User session ID
    %t - Date and time, in Common Log Format
    %u - Remote user that was authenticated (if any), else '-'
    %U - Requested URL path
    %v - Local server name
    %D - Time taken to process the request in millis. Note: In httpd %D is microseconds. Behaviour will be aligned to httpd in Tomcat 10 onwards.
    %T - Time taken to process the request, in seconds. Note: This value has millisecond resolution whereas in httpd it has second resolution. Behaviour will be align to httpd in Tomcat 10 onwards.
    %F - Time taken to commit the response, in millis
    %I - Current request thread name (can compare later with stacktraces)
    %X - Connection status when response is completed:
        X = Connection aborted before the response completed.
        + = Connection may be kept alive after the response is sent.
        - = Connection will be closed after the response is sent.

 There is also support to write information incoming or outgoing headers, cookies, session or request attributes and special timestamp formats. It is modeled after the Apache HTTP Server log configuration syntax. Each of them can be used multiple times with different xxx keys:

    %{xxx}i write value of incoming header with name xxx
    %{xxx}o write value of outgoing header with name xxx
    %{xxx}c write value of cookie with name xxx
    %{xxx}r write value of ServletRequest attribute with name xxx
    %{xxx}s write value of HttpSession attribute with name xxx
    %{xxx}p write local (server) port (xxx==local) or remote (client) port (xxx=remote)
    %{xxx}t write timestamp at the end of the request formatted using the enhanced SimpleDateFormat pattern xxx

All formats supported by SimpleDateFormat are allowed in %{xxx}t. In addition the following extensions have been added:

    sec - number of seconds since the epoch
    msec - number of milliseconds since the epoch
    msec_frac - millisecond fraction

These formats cannot be mixed with SimpleDateFormat formats in the same format token.

Furthermore one can define whether to log the timestamp for the request start time or the response finish time:

    begin or prefix begin: chooses the request start time
    end or prefix end: chooses the response finish time

By adding multiple %{xxx}t tokens to the pattern, one can also log both timestamps.

The shorthand pattern pattern="common" corresponds to the Common Log Format defined by '%h %l %u %t "%r" %s %b'.

The shorthand pattern pattern="combined" appends the values of the Referer and User-Agent headers, each in double quotes, to the common pattern.

    tomcat_juli_logging_level: "FINE"

Set [1catalina|2localhost|3manager|4host-manager].org.apache.juli.AsyncFileHandler.level and java.util.logging.ConsoleHandler.level to this loglevel. Possible values are:
  SEVERE, WARNING, INFO, CONFIG, FINE, FINER, FINEST or ALL. Default is FINE.

    tomcat_use_secure_flag: True

Set this attribute to True if you wish to have calls to request.isSecure() to return true for requests received by this Connector. You would want this on an SSL Connector or a non SSL connector that is receiving data from a SSL accelerator, like a crypto card, a SSL appliance or even a webserver. The default value is False.

    tomcat_session_http_only: True

Forcing Tomcat to use JSESSIONID cookie over only http.

    tomcat_session_secure: True

Forcing Tomcat to use secure JSESSIONID cookie.

    tomcat_manage_java_pkg: False

Tomcat manage java installation an install OpenJDK or not.

    tomcat_system_name: "tomcat_app_sys"

Optional: Use this folder name for this tomcat instance.

    tomcat_service_enabled: true

Enable or disable tomcat service on system startup.

    tomcat_system_home: "{{ tomcat_base_folder }}/{{ tomcat_system_user }}"

Folder of Tomcat binaries.

    # tomcat_system_home: "{{ tomcat_base_folder }}/{{ tomcat_system_name }}" # This is an optional setting to use tomcat_system_name for this system.

Folder of Tomcat binaries using tomcat_system_home variable.

    tomcat_catalina_home: "{{ tomcat_system_home }}/tomcat"

Tomcat Cataline home folder.

    tomcat_catalina_base: "{{ tomcat_catalina_home }}"

Tomcat Catalina base folder.

    tomcat_manage_firewalld: true

Role manages the firewalld settings of required ports.

    tomcat_manage_firewalld_use_zone: true

Tomcat firewalld uses zones (default) or use source addresses.

    tomcat_catalina_logs_directory_mode: "u=rwx,g=rwx,o="

Tomcat catalina logs directory mode.

    tomcat_java_version: 11

Configure Tomcat to use the specified version version of Java.

Tomcat LDAP authentication configuration:

    tomcat_ldap_enable: false

Enable Tomcat LDAP authentication. Disabled by default.

    tomcat_ldap_debug_level: 99

Tomcat LDAP authentication debug level. Default is 99.

    tomcat_ldap_url: 'ldap://ldap.cloud.department.ca:389'

Tomcat LDAP authentication URL. Do not use the default value. Tweak this value according your settings.

    tomcat_ldap_user: 'technicaluser@cloud.department.ca'

Tomcat LDAP user to reach LDAP server. Do not use the default value. Tweak this value according your settings.

    tomcat_ldap_pass: 'password'

Tomcat LDAP user's password to reach LDAP server. Do not use the default value. Tweak this value according your settings.

    tomcat_ldap_user_ou: 'ou=Users,dc=cloud,dc=department,dc=ca'

Organization Unit of valid users for Tomcat LDAP authentication. Tweak this value according your settings.

    tomcat_ldap_user_name: "(sAMAccountName={0})"

Name of authenticated users for Tomcat LDAP authentication. Default setting is (sAMAccountName={0}) which perfect for Windows Active Directory.

    tomcat_ldap_user_referrals: 'follow'



    tomcat_ldap_user_subtree: true

Tomcat LDAP user could be on the subtree of Organization Unit.

    tomcat_ldap_role_ou: 'ou=TomcatAdmin,ou=Groups,dc=cloud,dc=department,dc=ca'

Organization Unit of group of users for Tomcat LDAP authentication. Tweak this value according your settings. Create security groups here with the Tomcat role names like "manager-gui".

    tomcat_ldap_role_name: 'name'

User security group name as Tomcat role names like "manager-gui". Default is good if you want to use Tomcat role names.

    tomcat_ldap_role_subtree: true

Tomcat LDAP group could be on the subtree of Organization Unit.

    tomcat_ldap_role_search: '(member={0})'

Every member matching the security group's name could access the server as specified in the role.

## Dependencies

None.

## Example Playbook

    - hosts: all
      roles:
        - tomcat

## Licensing

The lactransformer application and documantations are licensed under the terms of
the MIT / BSD, you will find a copy of this license in the
[LICENSE](LICENSE) file included in the source package.

## Author Information

This role was created in 2016-2018 by Kálmán Szalai - KAMI

## Support

If you have any question, do not hesitate and drop me a line.
If you found a bug, or have a feature request, you can [fill an issue](https://github.com/KAMI911/ansible-role-tomcat/issues).

## Contributing

There are many ways to contribute to ansible-role-tomcat -- whether it be sending patches,
testing, reporting bugs, or reviewing and updating the documentation. Every
contribution is appreciated!

Please continue reading in the [contributing chapter](CONTRIBUTING.md).

### Fork me on Github

https://github.com/KAMI911/ansible-role-tomcat

Add a new remote `upstream` with this repository as value.

```
git remote add upstream https://github.com/KAMI911/ansible-role-tomcat.git
```

You can pull updates to your fork's master branch:

```
git fetch --all
git pull upstream HEAD
```

## Donation

If you find this useful, please consider a donation:

[![paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=RLQZ58B26XSLA)

<!-- TOC URLs -->
[Requirements]: #requirements
[Installation]: #installation
[Role Variables]: #role_variables
[Dependencies]: #dependencies
[Example Playbook]: #example_playbook
[Licensing]: #licensing
[Author Information]: #author_information
[Support]: #support
[Contributing]: #contributing
[Donation]: #donation
