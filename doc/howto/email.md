# Email

## Sending notification emails via SMTP

GDK is by default not prepared to actually send emails via SMTP; this only
applies to production mode. In development mode, GDK uses `letter_opener_web`
to show sent messages in a web interface under
`http://localhost:3000/rails/letter_opener`.

In order to enable SMTP delivery:

1. You will need an SMTP-capable account. An option is to create a dummy account
   on Gmail for this purpose.
1. Copy `gitlab/config/initializers/smtp_settings.rb.sample` to `smtp_settings.rb`
   and configure the SMTP connection details (see the
   [Omnibus SMTP documentation](https://docs.gitlab.com/omnibus/settings/smtp.html)):

   ```ruby
   # if Rails.env.production?
   Rails.application.config.action_mailer.delivery_method = :smtp

   ActionMailer::Base.delivery_method = :smtp
   ActionMailer::Base.smtp_settings = {
     address: 'smtp.gmail.com',
     port: 587,
     user_name: 'foobar@gmail.com',
     password: 'my-password',
     domain: 'smtp.gmail.com',
     authentication: :login,
     enable_starttls_auto: true,
     tls: false,
     openssl_verify_mode: 'peer' # See ActionMailer documentation for other possible options,
   }
   # end
   ```

   - Commenting out the `Rails.env.production?` conditional allows us to use
     the configuration with the `development` Rails environment.
   - In the sample Gmail is used as SMTP provider (you need to enable 2FA, then
     create an application password).
1. In `gitlab/config/environments/development.rb` make sure that ActionMailer
   logs delivery errors, this will help you troubleshoot SMTP delivery:

   ```ruby
   config.action_mailer.raise_delivery_errors = true
   ```

1. Then test the configuration is correctly loaded and try to deliver a message:

   ```shell
   bin/rails console

   [1] pry(main)> ActionMailer::Base.smtp_settings
   => {:address=>"smtp.gmail.com",
    :port=>587,
    :user_name=>"foobar@gmail.com",
    :password=>"my-password",
    :domain=>"smtp.gmail.com",
    :authentication=>:login,
    :enable_starttls_auto=>true,
    :tls=>false,
    :openssl_verify_mode=>"peer"}

   [2] pry(main)> ActionMailer::Base.delivery_method
   => :smtp

   [3] pry(main)> Notify.test_email('foobar@gmail.com', 'Hello World', 'This is a test message').deliver_now
   ...
   => #<Mail::Message:70169718084300, Multipart: false, Headers: <Date: Thu, 11 Jul 2019 16:59:49 +0200>,
      <From: GitLab <example@example.com>>, <Reply-To: GitLab <noreply@example.com>>,
      <To: foobar@gmail.com>, <Message-ID: <5d274ee5122c6_57a3fd1a782dfd03593e@macos.local.mail>>,
      <Subject: Hello World>, <Mime-Version: 1.0>, <Content-Type: text/html; charset=UTF-8>,
      <Content-Transfer-Encoding: 7bit>, <Auto-Submitted: auto-generated>,
      <X-Auto-Response-Suppress: All>>
   ```

### macOS OpenSSL-specific configuration

If you're using macOS, there's
[known issues with the OpenSSL certificates in macOS Ruby](https://gitlab.com/gitlab-org/gitlab/issues/13914),
a workaround is to:

1. Install OpenSSL with brew:

   ```shell
   brew install openssl
   ```

1. Make sure to include the actual `ca_path` and `ca_file` configurations in the
   `smtp_settings.rb` file above, or when sending the emails you will be
   getting `SSLError` issues in the GitLab logs:

   ```ruby
   ActionMailer::Base.smtp_settings = {
     # ...
     ca_path: '/usr/local/etc/openssl',
     ca_file: '/usr/local/etc/openssl/cert.pem'
   }
   ```

## Enabling S/MIME delivery

You can follow the [official S/MIME emails documentation](https://docs.gitlab.com/ce/administration/smime_signing_email.html)
and combine it with the SMTP configuration above to test actual delivery of
signed messages.

If you do not have an S/MIME key pair for testing, you can either create your
own self-signed one, or purchase one. MozillaZine keeps a nice collection
of [S/MIME-capable signing authorities](http://kb.mozillazine.org/Getting_an_SMIME_certificate)
and some of them generate keys for free.

