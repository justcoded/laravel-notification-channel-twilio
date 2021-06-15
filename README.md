# Twilio notifications channel for Laravel

## Contents

- [Installation](#installation)
- [Usage](#usage)
    - [Available Message methods](#available-message-methods)

## Installation

You can install the package via composer:

``` bash
composer require justcoded/laravel-notification-channel-twilio
```

### Configuration

Add your Twilio Account SID, Auth Token, and From Number (optional) to your `.env`:

```dotenv
TWILIO_USERNAME=XYZ # optional when using auth token
TWILIO_PASSWORD=ZYX # optional when using auth token
TWILIO_AUTH_TOKEN=ABCD # optional when using username and password
TWILIO_ACCOUNT_SID=1234 # always required
TWILIO_FROM=100000000 # optional default from
TWILIO_ALPHA_SENDER=HELLO # optional
TWILIO_DEBUG_TO=23423423423 # Set a number that call calls/messages should be routed to for debugging
TWILIO_SMS_SERVICE_SID=MG0a0aaaaaa00aa00a00a000a00000a00a # Optional but recommended 
```

### Advanced configuration

Run `php artisan vendor:publish --provider="Justcoded\NotificationChannelTwilio\TwilioProvider"`
```
/config/twilio-notification-channel.php
```

#### Suppressing specific errors or all errors

Publish the config using the above command, and edit the `ignored_error_codes` array. You can get the list of
exception codes from [the documentation](https://www.twilio.com/docs/api/errors).

If you want to suppress all errors, you can set the option to `['*']`. The errors will not be logged but notification
failed events will still be emitted.

#### Recommended Configuration

Twilio recommends always using a [Messaging Service](https://www.twilio.com/docs/sms/services) because it gives you
access to features like Advanced Opt-Out, Sticky Sender, Scaler, Geomatch, Shortcode Reroute, and Smart Encoding.

Having issues with SMS? Check Twilio's [best practices](https://www.twilio.com/docs/sms/services/services-best-practices).

## Upgrading from 2.x to 3.x

If you're upgrading from version `2.x`, you'll need to make sure that your set environment variables match those above
in the config section. None of the environment variable names have changed, but if you used different keys in your
`services.php` config then you'll need to update them to match the above, or publish the config file and change the
`env` key.

You should also remove the old entry for `twilio` from your `services.php` config, since it's no longer used.

The main breaking change between `2.x` and `3.x` is that failed notification will now throw an exception unless they are
in the list of ignored error codes (publish the config file to edit these).

You can replicate the `2.x` behaviour by setting `'ignored_error_codes' => ['*']`, which will case all exceptions to be
suppressed.

## Usage

Now you can use the channel in your `via()` method inside the notification:

``` php
use Justcoded\NotificationChannelTwilio\TwilioChannel;
use Justcoded\NotificationChannelTwilio\TwilioSmsMessage;
use Illuminate\Notifications\Notification;

class AccountApproved extends Notification
{
    public function via($notifiable)
    {
        return [TwilioChannel::class];
    }

    public function toTwilio($notifiable)
    {
        return (new TwilioSmsMessage())
            ->content("Your {$notifiable->service} account was approved!");
    }
}
```

You can also send an MMS:

``` php
use Justcoded\NotificationChannelTwilio\TwilioChannel;
use Justcoded\NotificationChannelTwilio\TwilioMmsMessage;
use Illuminate\Notifications\Notification;

class AccountApproved extends Notification
{
    public function via($notifiable)
    {
        return [TwilioChannel::class];
    }

    public function toTwilio($notifiable)
    {
        return (new TwilioMmsMessage())
            ->content("Your {$notifiable->service} account was approved!")
            ->mediaUrl("https://picsum.photos/300");
    }
}
```

Or create a Twilio call:

``` php
use Justcoded\NotificationChannelTwilio\TwilioChannel;
use Justcoded\NotificationChannelTwilio\TwilioCallMessage;
use Illuminate\Notifications\Notification;

class AccountApproved extends Notification
{
    public function via($notifiable)
    {
        return [TwilioChannel::class];
    }

    public function toTwilio($notifiable)
    {
        return (new TwilioCallMessage())
            ->url("http://example.com/your-twiml-url");
    }
}
```

In order to let your Notification know which phone are you sending/calling to, the channel will look for the `phone_number` attribute of the Notifiable model. If you want to override this behaviour, add the `routeNotificationForTwilio` method to your Notifiable model.

```php
public function routeNotificationForTwilio()
{
    return '+1234567890';
}
```

### Available Message methods

#### TwilioSmsMessage

- `from('')`: Accepts a phone to use as the notification sender.
- `content('')`: Accepts a string value for the notification body.
- `messagingServiceSid('')`: Accepts a messaging service SID to handle configuration.

#### TwilioCallMessage

- `from('')`: Accepts a phone to use as the notification sender.
- `url('')`: Accepts an url for the call TwiML.
