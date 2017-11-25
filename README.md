vmail - Virtual Mail Hosting
============================

`vmail` is a script and a configuration of Postfix + Dovecot that
makes it simple and easy to manage your own mail server, for one
or more custom domains.  It supports aliasing, multiple virtual
domains, and mailing lists.

Server Configuration
--------------------

To configure a server to be a vmail-powered mail server:

```sh
vmail init --domain root.domain
```

This will install the necessary software (Postfix / Dovecot),
write the appropriate configuration files, and set the system up
to start the mail server daemons on boot.

It is safe to run this command on a server that has already been
initialized to run vmail.

To check the current configuration, and to view some summary and
status information about domains, mailing lists, mailboxes, etc:

```sh
vmail status
```

Email Domains
-------------

To add a new domain to your mail server:

```sh
vmail add domain mail.doma.in
```

To list the currently recognized domain:

```sh
vmail ls domain
```

Mailboxes
---------

To list all defined mailboxes, across all domains:

```sh
vmail ls mbox
```

To limit results to a single domain:

```sh
vmail ls mbox @mail.doma.in
```

To limit to a single mailbox name, across all domains:

```sh
vmail ls mbox james@
```

Each mailbox represents a singular destination for email.
Mailboxes may have multiple aliases, but in order to retreive
messages via IMAP or POP3. the owner of the mailbox needs to
authenticate with the canonical mailbox name and the password they
set.

To change the password (interactively) for a mailbox:

```sh
vmail passwd user@mail.doma.in
```

(No knowledge of the old password is required, but this command
can only be run from the mail server shell itself, so...)

To create a new mailbox, any of the following will suffice:

```sh
vmail add mbox user@mail.doma.in
vmail add mbox user@mail.doma.in "Display Name"
vmail add mbox user@mail.doma.in "Display Name" secret-password
```

To remove a mailbox (warning: destructive):

```sh
vmail rm mbox user@mail.doma.in
```

Email Aliases
-------------

You can set up an unlimited number of aliases for your mailboxes,
either within the same domain or across hosted domains.  Mail sent
to an alias will be delivered to that aliased mailbox.

To create a new alias:

```sh
vmail add alias alias@mail.doma.in mailbox
vmail add alias alias@mail.doma.in mailbox@other.doma.in
```

To list all aliases, across all domains:

```sh
vmail ls alias
```

To limit the results to aliases from or to a given domain:

```sh
vmail ls alias @mail.doma.in
```

To limit the results to users from or to a given user, across
domains:

```sh
vmail ls alias user@
```

To limit the result to a single address (either as the alias, or
as the destination mailbox):

```sh
vmail ls alias user@mail.doma.in
```

To remove an alias:

```sh
vmail rm alias alias@mail.doma.in
```

Mailing Lists
-------------

A mailing list is a collection of email addresses, termed
_subscribers_, who receive copies of messages sent to the list
address.  This can be accomplished manually by defining aliases in
your Postfix configuration, but that is a time-consuming approach
that places the burden of list maintenance squarely on the
shoulders of the email server administrator.

With `vmail`, subscribers can subscribe to or unsubscribe from the
list on their own, without the assistance of the email server
administrator or the list owner.  To subscribe to a list, users
need only send an email with the word "subscribe" in either the
subject or the first few lines of the body, to the list address.
To unsubscribe, a message with the word "unsubscribe" is sent
instead.

These _control_ messages are not passed along to the list
subscribers.  Instead, a confirmation email, with an hard-to-guess
token, is sent back to the sender, and if they reply to it, their
subscription status change will be processed.

Savvy mailing list subscribers may also use the `<list>-request`
email address to submit and process these requests; the abililty
for the main list address to also process them is merely a
convenience to catch common behavior of less worldly-wise
subscribers.

To list all of your mailing lists:

```sh
vmail ls lists
```

To limit results to a single domain:

```sh
vmail ls lists @mail.doma.in
```

To create a new mailing list:

```sh
vmail add list \
  --name "My New Mailing List" \
  --owner user@mail.doma.in \
  --archives /srv/mail/www/archives/list \
  list@mail.doma.in
```

`vmail` **requires** that all mailing lists be hosted on a domain
that starts with `lists.`.  To implement mailing lists, vmail
(ab)uses Postfix transports to provide a simple and reliable means
of piping inbound email messages to the mailing list manager
program (also part of `vmail`).  If you try to create a list on a
domain that doesn't start with `lists.`, vmail will create an
alias on the virtual domain you specified, pointing to the list
domain.  Any new domains will be automagically created for you.

For example, from a fresh vmail installation, trying to create the
list `users@example.com`, will result in the following
configuration tasks:

  1. The virtual domain `example.com` will be created.
  2. The list domain `lists.example.com` will be created.
  3. The dummy account `users@lists.example.com` will be created.
  4. The alias `users@example.com` will be created, pointing at
     `users@lists.example.com`.

(The dummy account is necessary to convince Postfix that the
list exists and that it should accept mail RCPT TO it.)

If configured to do so (via the `--archives` flag) vmail will save
copies of all messages sent to the list to an _archive_, in HTML
format, which can then be viewed via a web browser.  Raw mbox
archives will also be created, for backup and forensic purposes.

To view a mailing list's subscribers:

```sh
vmail ls subs list@mail.doma.in
```

To manually subscribe someone to the list, without confirmation:

```sh
vmail sub list@mail.doma.in user@other.doma.in
```

(order doesn't matter; as long as one address is a list or an
alias to a list, and the other is not, vmail does the right
thing).

To manually unsubscribe someone from the list, without
confirmation:

```sh
vmail unsub list@mail.doma.in user@other.doma.in
```

(again, order does not matter.)

To block an account, preventing the owner from re-subscribing to
the list.  Blocked accounts will be automatically unsubscribed,
and future requests from that sender will be ignored.

```sh
vmail block list@mail.doma.in user@other.doma.in
```

To unblock a previously blocked account:

```sh
vmail unblock list@mail.doma.in user@other.doma.in
```

Unblocked users will not automatically be re-subscribed.

Contributing
------------

I open sourced vmail, after having used it in production for close
to 5 years with no issues.  I hope you find it useful.  All
discussion related to this project is handled via the mailing
list, [vmail@lists.jameshunt.us][sub].  If you find a bug,
aberrant behavior, or missing feature, please discuss there
before filing a bug report or submitting a patch.



[sub]: mailto:vmail-request@lists.jameshunt.us?subject=subscribe
