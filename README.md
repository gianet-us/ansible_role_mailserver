# **Ansible Role: Postfix**

An Ansible role to install and configure Postfix on Debian-based systems.

## **Description**

This role sets up Postfix to function as a local mail server designed for internal use. Its primary function is to accept mail from local services and relay all outbound messages through a configured **smarthost**.

This is the perfect setup for environments where internal applications (like cron, monitoring systems, or web applications) need to send email notifications without the complexity of managing a full, internet-facing mail server.

This role performs the following actions:

* Installs the Postfix package and necessary SASL modules on Debian/Ubuntu.  
* Manages the main Postfix configuration file (/etc/postfix/main.cf) via a template.  
* Manages the /etc/mailname file for defining the mail domain.  
* Configures Postfix to route all outgoing mail through a specified smarthost.  
* Securely configures SASL authentication for the smarthost if credentials are provided.

## **Requirements**

* **Target OS**: This role is designed exclusively for **Debian-based** distributions (e.g., Debian, Ubuntu).  
* **Ansible**: Version 2.10 or newer.

## **Role Variables**

The role's behavior can be customized using the following variables. The default values are defined in defaults/main.yml.

| Variable | Default Value | Description |
| :---- | :---- | :---- |
| postfix_relayhost | "" (empty string) | **Required.** The smarthost for relaying all mail. Use square brackets [] to prevent MX lookups (e.g., \[smtp.sendgrid.net\]:587). |
| postfix_relayhost_user | (undefined) | The username for SASL authentication with the smarthost. If defined with a password, SASL auth will be enabled. |
| postfix_relayhost_password | (undefined) | The password or API key for the smarthost user. **It** is strongly recommended to store this in Ansible **Vault.** |
| postfix_mail_domain | `{{ ansible_domain \| default('internal.local') }}` | The primary mail domain for this server |
| postfix_myhostname | `mail.{{ postfix_mail_domain }}` | The fully qualified domain name (FQDN) of the mail server itself (e.g., mail.example.com). |
| postfix_mydestination | `$myhostname, localhost.{{ postfix_mail_domain }}, localhost, {{ postfix_mail_domain }}` | A comma-separated list of domains this server will accept mail for. The default is usually sufficient for an internal relay. |
| postfix_mynetworks | `"127.0.0.0/8 [::1]/128"` | The list of "trusted" remote SMTP clients that have more privileges than "strangers"|
| postfix_inet_interfaces | all | The network interfaces Postfix listens on. Set to loopback-only to only accept mail from the server itself. |
| postfix_inet_protocols | all | The IP protocols to use (ipv4, ipv6, or all). |

### **Dovecot Configuration**

The role now supports installing and configuring Dovecot for IMAP/POP3 services.

| Variable | Default Value | Description |
| :---- | :---- | :---- |
| dovecot_enabled | true | Whether to install and configure Dovecot. |
| dovecot_protocols | "imap pop3 lmtp" | Protocols to enable. |
| dovecot_mail_location | "maildir:~/Maildir" | Mail storage location. |
| dovecot_ssl | "yes" | SSL/TLS configuration (yes, no, required). |
| dovecot_ssl_cert | snakeoil | Path to SSL certificate. |
| dovecot_ssl_key | snakeoil | Path to SSL key. |
| dovecot_auth_mechanisms | "plain login" | Authentication mechanisms. |
| dovecot_postfix_sasl_enable | true | Enable Postfix SASL authentication via Dovecot. |
| dovecot_postfix_lmtp_enable | true | Enable Postfix delivery via Dovecot LMTP. |

### **SASL Authentication**

SASL authentication for the smarthost is **automatically enabled** if both postfix_relayhost_user and postfix_relayhost_password are defined. If they are not defined, Postfix will attempt to send mail without authentication.

## **Dependencies**

This role has no dependencies on other Ansible roles or collections beyond the standard ansible.builtin modules.

## **Example Playbook**

Here is a basic example of how to use this role in your playbook. You must define postfix_relayhost. It is also highly recommended to use Ansible Vault to encrypt the smarthost password.

```
---  
- hosts: all  
  become: true  
  roles:  
    - role: your_username.postfix
      vars:  
        postfix_relayhost: "[smtp.mailgun.org\]:587"  
        postfix_relayhost_user: "postmaster@mg.example.com"  
        postfix_relayhost_password: "{{ vaulted_mailgun_password }}" # Stored in Ansible Vault  
        postfix_inet_interfaces: "loopback-only"  
        postfix_mail_domain: "example.com"
```

## **License**

GPL-3.0-only

## **Author Information**

This role was created by Giacchetta Networks.