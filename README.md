# Ansible Role: Mail

An Ansible role to deploy and configure an internal mail server on Debian-based systems.

## Description

This role sets up a complete internal mail server stack, currently including:

- **Postfix** - Mail Transfer Agent (MTA) for sending and receiving mail
- **Dovecot** - IMAP/POP3 server for mail retrieval

The role is designed for **internal use cases** where applications, services, and users within your infrastructure need to send and receive email. Outbound mail is relayed through a configured smarthost (e.g., SendGrid, Mailgun, or your ISP's SMTP server).

### Use Cases

- Internal applications sending notifications (cron jobs, monitoring, CI/CD pipelines)
- Service accounts that need to receive and process email
- Development and testing environments
- Private mail infrastructure for small teams

### What This Role Does NOT Include

This role intentionally omits antispam and antivirus components. Since it's designed for internal mail that doesn't interact with external/untrusted sources, these features are unnecessary and would add complexity.

## Requirements

- **Target OS**: Debian-based distributions (Debian, Ubuntu)
- **Ansible**: Version 2.10 or newer

## Role Variables

Default values are defined in `defaults/main.yml`.

### General Settings

| Variable | Default | Description |
| :--- | :--- | :--- |
| mail_ssl_cert | snakeoil | Path to SSL certificate (shared by Postfix and Dovecot). |
| mail_ssl_key | snakeoil | Path to SSL private key (shared by Postfix and Dovecot). |

### Postfix Configuration

| Variable | Default | Description |
| :--- | :--- | :--- |
| postfix_relayhost | "" | **Required.** Smarthost for relaying outbound mail. Use brackets to skip MX lookups (e.g., `[smtp.sendgrid.net]:587`). |
| postfix_relayhost_user | (undefined) | Username for smarthost SASL authentication. |
| postfix_relayhost_password | (undefined) | Password/API key for smarthost. Store in Ansible Vault. |
| postfix_mail_domain | `{{ ansible_domain }}` | Primary mail domain for this server. |
| postfix_myhostname | `mail.{{ postfix_mail_domain }}` | FQDN of the mail server. |
| postfix_mydestination | `$myhostname, localhost...` | Domains accepted for local delivery. |
| postfix_mynetworks | `127.0.0.0/8 [::1]/128` | Trusted networks allowed to relay. |
| postfix_inet_interfaces | all | Network interfaces to listen on. Use `loopback-only` for local-only access. |
| postfix_inet_protocols | all | IP protocols to use (ipv4, ipv6, or all). |

SASL authentication for the smarthost is automatically enabled when both `postfix_relayhost_user` and `postfix_relayhost_password` are defined.

### Dovecot Configuration

| Variable | Default | Description |
| :--- | :--- | :--- |
| dovecot_enabled | true | Install and configure Dovecot. |
| dovecot_protocols | "imap pop3 lmtp" | Protocols to enable. |
| dovecot_mail_location | "maildir:~/Maildir" | Mail storage format and location. |
| dovecot_ssl | "yes" | SSL/TLS mode: `yes`, `no`, or `required`. |
| dovecot_auth_mechanisms | "plain login" | Allowed authentication mechanisms. |
| dovecot_postfix_sasl_enable | true | Allow Postfix to authenticate users via Dovecot. |
| dovecot_postfix_lmtp_enable | true | Deliver mail to Dovecot via LMTP. |
| dovecot_imap_capability | "" | Adjust advertised IMAP capabilities (e.g., `+IMAP4rev1 -LITERAL+`). |
| dovecot_users | [] | List of virtual mailbox users. See below. |

### Virtual Mailbox Users

Define users for Dovecot virtual mailboxes:

```yaml
dovecot_users:
  - name: "service1"
    pass: "mysecretpassword"
```

For security, the role generates a random 16-character token on the server (stored in `/etc/dovecot/dovecot_token`). The actual password is `token + password`. For example, if the token is `He5rN5SPH33AbFLn`, the user must authenticate with `He5rN5SPH33AbFLnmysecretpassword`.

## Dependencies

None.

## Example Playbook

```yaml
---
- hosts: mail_servers
  become: true
  roles:
    - role: giacchetta.mail
      vars:
        postfix_mail_domain: "example.com"
        postfix_relayhost: "[smtp.mailgun.org]:587"
        postfix_relayhost_user: "postmaster@mg.example.com"
        postfix_relayhost_password: "{{ vault_mailgun_password }}"
        mail_ssl_cert: "/etc/letsencrypt/live/mail.example.com/fullchain.pem"
        mail_ssl_key: "/etc/letsencrypt/live/mail.example.com/privkey.pem"
        dovecot_ssl: "required"
        dovecot_users:
          - name: "alerts"
            pass: "{{ vault_alerts_password }}"
```

## License

GPL-3.0-only

## Author Information

This role was created by Giacchetta Networks.
