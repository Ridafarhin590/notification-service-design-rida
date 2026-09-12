# Notification Service System --- Complete ERD Design

## 1. Purpose

This document defines a complete Entity Relationship Diagram (ERD) for a production-oriented Notification Service.

The design supports:

-   Email notifications

-   SMS notifications

-   Push notifications

-   In-App notifications

-   Web/browser notifications

-   WhatsApp notifications

-   Voice/call notifications

-   Scheduled notifications

-   Bulk notifications

-   Notification templates

-   User notification preferences

-   Multiple recipients

-   Multiple delivery channels for one notification

-   Delivery tracking

-   Retry attempts

-   Provider configuration

-   Webhook/callback tracking

-   Device/token management

-   Notification categories and events

-   Quiet hours and opt-out preferences

-   Audit/history

-   Dead-letter handling

> The database design is intentionally normalized. 

> A logical notification is separated from its individual channel deliveries so that one notification can succeed on one channel and fail on another.

------------------------------------------------------------------------

# 2. Supported Notification Types

## 2.1 Email

Examples:

-   Welcome email

-   Password reset

-   Order confirmation

-   Invoice

-   Security alert

-   Marketing campaign

Typical provider examples:

-   SMTP

-   SendGrid

-   Amazon SES

-   Mailgun

------------------------------------------------------------------------

## 2.2 SMS

Examples:

-   OTP

-   Transaction alert

-   Delivery update

-   Security notification

Typical provider examples:

-   Twilio

-   AWS SNS

-   Vonage

------------------------------------------------------------------------

## 2.3 Push Notification

Used for Android, iOS, and web applications.

Examples:

-   New message

-   Order status

-   Payment confirmation

-   Promotional alert

Typical provider:

-   Firebase Cloud Messaging (FCM)

-   Apple Push Notification service (APNs)

------------------------------------------------------------------------

## 2.4 In-App Notification

Notifications displayed inside the application.

Examples:

-   "Your order has shipped"

-   "You have a new message"

-   "Your profile was updated"

These notifications are usually stored in the database and read by the frontend.

------------------------------------------------------------------------

## 2.5 Web/Browser Notification

Browser notifications can be sent to users who have granted browser notification permission.

A browser subscription/token is associated with a user/device.

------------------------------------------------------------------------

## 2.6 WhatsApp

Can be used for:

-   Order updates

-   OTP

-   Customer support

-   Transaction notifications

-   Marketing messages

Usually implemented through an approved WhatsApp Business provider/API.

------------------------------------------------------------------------

## 2.7 Voice Notification

A voice call can be triggered for high-priority notifications.

Examples:

-   Critical security alerts

-   Fraud alerts

-   Infrastructure incidents

------------------------------------------------------------------------
## 3. Complete ERD

![Notification Service Complete ERD](./Notification_Service_Complete_ERD.png)


------------------------------------------------------------------------

# 4. Core Entities

The core model is:


USER

  |
  v

NOTIFICATION

  |
  +---- RECIPIENT
  |
  +---- DELIVERY
          |
          +---- CHANNEL
          |
          +---- PROVIDER
          |
          +---- RETRY ATTEMPTS
          |
          +---- WEBHOOK EVENTS

Additional configuration:

USER

 |
 +---- NOTIFICATION PREFERENCES
 |
 +---- CHANNEL PREFERENCES
 |
 +---- DEVICES
 |
 +---- QUIET HOURS

------------------------------------------------------------------------

# 5. USERS

Stores application users.

  Column             Type        Key      Description

  ------------------ ----------- -------- ------------------------

  id                 BIGINT      PK       Internal user ID

  external_user_id   VARCHAR     UNIQUE   ID from another system

  name               VARCHAR              User name

  email              VARCHAR     UNIQUE   Email address

  phone              VARCHAR     UNIQUE   Phone number

  status             VARCHAR              ACTIVE/INACTIVE

  created_at         TIMESTAMP            Creation time

  updated_at         TIMESTAMP            Last update



Relationship:


USERS 1 ---- N NOTIFICATIONS

USERS 1 ---- 1 USER_NOTIFICATION_PREFERENCES

USERS 1 ---- N USER_DEVICES

USERS 1 ---- N USER_CHANNEL_PREFERENCES

USERS 1 ---- N USER_QUIET_HOURS

------------------------------------------------------------------------

# 6. USER_DEVICES

Stores mobile/browser devices used for push notifications.

Example:


User

 |

 +-- Android device

 |

 +-- iPhone

 |

 +-- Chrome browser


  Column         Type        Key

  -------------- ----------- -----

  id             BIGINT      PK

  user_id        BIGINT      FK

  device_type    VARCHAR    

  device_token   VARCHAR    

  platform       VARCHAR    

  browser        VARCHAR    

  active         BOOLEAN    

  last_seen_at   TIMESTAMP  

  created_at     TIMESTAMP  

This entity is particularly important for Push and Web notifications.

------------------------------------------------------------------------

# 7. USER_NOTIFICATION_PREFERENCES

Stores global notification preferences.

Example:

text

All Notifications    = TRUE

Marketing            = FALSE

Transactional        = TRUE

Security             = TRUE

Relationship:


USERS 1 ---- 1 USER_NOTIFICATION_PREFERENCES

------------------------------------------------------------------------

# 8. USER_CHANNEL_PREFERENCES

Stores channel-specific preferences.

Example:

text

EMAIL    -> enabled

SMS      -> enabled

PUSH     -> enabled

WHATSAPP -> disabled


It can also store the preferred destination.

  Column       Description

  ------------ ----------------------------

  user_id      User

  channel_id   Notification channel

  enabled      Whether channel is enabled

  address      Email/phone/address

  priority     Preferred channel priority

------------------------------------------------------------------------

# 9. USER_QUIET_HOURS

Allows users to specify periods during which non-critical notifications

should not be delivered.

Example:

text

22:00 → 07:00

Timezone → Asia/Kolkata

Critical/security notifications can be configured to bypass quiet hours.

------------------------------------------------------------------------

# 10. NOTIFICATION_CATEGORIES

Groups notification types.

Examples:

text

TRANSACTIONAL

SECURITY

MARKETING

SYSTEM

SOCIAL

ORDER

PAYMENT

Example:


SECURITY

   |

   +-- LOGIN_ALERT

   +-- PASSWORD_CHANGED

   +-- SUSPICIOUS_ACTIVITY

------------------------------------------------------------------------

# 11. NOTIFICATION_TYPES

Defines the business event/type.

Examples:


ORDER_CREATED

ORDER_SHIPPED

PAYMENT_SUCCESS

PASSWORD_RESET

OTP

LOGIN_ALERT

WELCOME

PROMOTION

Relationship:

NOTIFICATION_CATEGORIES 1 ---- N NOTIFICATION_TYPES


------------------------------------------------------------------------

# 12. NOTIFICATION_TEMPLATES

Stores reusable notification content.

Example:

template_key = ORDER_CONFIRMATION

channel = EMAIL

language = en

version = 2

Subject:

Order Confirmed

Content:

Hello {{name}}, your order {{orderId}} has been confirmed.


The same business notification can have different templates:


ORDER_CONFIRMATION

 |

 +-- EMAIL template

 +-- SMS template

 +-- PUSH template

 +-- WHATSAPP template



------------------------------------------------------------------------

# 13. NOTIFICATIONS

This is the central logical notification entity.

It represents the business notification rather than a particular

delivery attempt.

Example:

text

Notification ID: 1001

Type: ORDER_CREATED

Title: Order Confirmed

Message: Your order #1234 has been confirmed.

Priority: HIGH

Status: QUEUED



Possible statuses:


PENDING

QUEUED

PROCESSING

PARTIALLY_SENT

SENT

FAILED

CANCELLED

SCHEDULED


Important distinction:

`NOTIFICATIONS.status\` represents the overall logical notification.

`NOTIFICATION_DELIVERIES.status\` represents the result for an individual channel.

------------------------------------------------------------------------

# 14. NOTIFICATION_RECIPIENTS**

Separating recipients from notifications allows the system to support:

-   Multiple recipients

-   To/CC/BCC

-   Group notifications

-   Direct destinations

-   Users who are not registered

Example:

text

Notification #1001

 |

 +-- User A

 +-- User B

 +-- User C


Recipient types can include:

text

TO

CC

BCC



------------------------------------------------------------------------

# 15. NOTIFICATION_CHANNELS

Defines the supported delivery channels.

Recommended records:


EMAIL

SMS

PUSH

IN_APP

WEB_PUSH

WHATSAPP

VOICE


Example:

    id name       type       active

  ---- ---------- ---------- --------

     1 Email      EMAIL      true

     2 SMS        SMS        true

     3 Push       PUSH       true

     4 In-App     IN_APP     true

     5 Web Push   WEB_PUSH   true

     6 WhatsApp   WHATSAPP   true

     7 Voice      VOICE      true

------------------------------------------------------------------------

# 16. NOTIFICATION_PROVIDERS

A channel can have multiple providers.

Example:

text

EMAIL

 |

 +-- SMTP

 +-- Amazon SES

 +-- SendGrid


Another example:

text

SMS

 |

 +-- Twilio

 +-- AWS SNS



This allows provider fallback.

Example:

 text

Primary SMS Provider

       |

       X failure

       |

       v

Secondary SMS Provider



------------------------------------------------------------------------

# 17. NOTIFICATION_DELIVERIES

This is the most important table for multi-channel delivery.

One logical notification can create multiple deliveries.

Example:

text

Notification #1001

 |

 +-- Email delivery  -\> SENT

 |

 +-- SMS delivery    -\> FAILED

 |

 +-- Push delivery   -\> SENT

 |

 +-- WhatsApp        -\> SENT



This means the system does not lose channel-level delivery information.

Possible statuses:

text

PENDING

QUEUED

PROCESSING

SENT

DELIVERED

FAILED

RETRYING

BOUNCED

REJECTED

CANCELLED


------------------------------------------------------------------------

# 18. NOTIFICATION_RETRY_ATTEMPTS

Tracks each attempt separately.

Example:

text

Delivery #5001

Attempt 1 -> FAILED

Attempt 2 -> FAILED

Attempt 3 -> SUCCESS


Example data:

    attempt status    error

  --------- --------- ----------------

          1 FAILED    TIMEOUT

          2 FAILED    PROVIDER_ERROR

          3 SUCCESS   NULL

This gives a complete failure history.

------------------------------------------------------------------------

# 19. NOTIFICATION_WEBHOOKS

External providers may asynchronously notify the application.

Example:


Provider

   |
   | webhook
   v

Notification Service



Webhook events may include:

text

SENT

DELIVERED

BOUNCED

FAILED

READ

CLICKED


This is particularly useful for email, SMS, WhatsApp, and push providers that support delivery callbacks.

------------------------------------------------------------------------

# 20. NOTIFICATION_EVENTS

Provides an audit trail for notification state changes.

Example:


PENDING

   ↓

QUEUED

   ↓

PROCESSING

   ↓

 SENT

   ↓

DELIVERED

   ↓

READ


Each transition can be stored as an event.

------------------------------------------------------------------------

# 21. NOTIFICATION_ATTACHMENTS**

Used for email or other channels that support attachments.

Examples:

Invoice.pdf

Receipt.pdf

Report.xlsx

Welcome.pdf



Relationship:


NOTIFICATIONS 1 ---- N NOTIFICATION_ATTACHMENTS


------------------------------------------------------------------------

# 22. NOTIFICATION_CAMPAIGNS**

Used for bulk/marketing notifications.

Example:

Campaign:

"Diwali Sale"

Recipients:

100,000 users

Channels:

EMAIL + PUSH


Relationship:


NOTIFICATION_CAMPAIGNS 1 ---- N NOTIFICATIONS


A campaign can create many logical notifications.

------------------------------------------------------------------------

# 23. DEAD_LETTER_NOTIFICATIONS

If a notification repeatedly fails after the configured retry limit, it

can be moved to a dead-letter table/queue for investigation or later

reprocessing.

Example:

text

Delivery

   ↓

Retry 1

   ↓

Retry 2

   ↓

Retry 3

   ↓

FAILED

   ↓

Dead Letter


------------------------------------------------------------------------

# 24. Email Notification Flow

Application

    |
    v

Notification Service

    |
    v

NOTIFICATIONS

    |
    v

Check Email Preference

    |
    v

EMAIL DELIVERY

    |
    v

Email Provider

    |
    v

Webhook

    |
    v

DELIVERED



Database relationship:


USERS

  |
  v

NOTIFICATIONS

  |
  v

NOTIFICATION_DELIVERIES

  |

  +--> EMAIL CHANNEL

  |

  +--> EMAIL PROVIDER



------------------------------------------------------------------------

# 25. SMS Notification Flow**


Application

    |
    v

Notification Service

    |
    v

Check SMS Preference

    |
    v

SMS DELIVERY

    |
    v

SMS Provider

    |
    v

Webhook

    |
    v

DELIVERED



------------------------------------------------------------------------

# 26. Push Notification Flow**


User

 |

 +--> USER_DEVICES

          |
          v

     Device Token

          |
          v

NOTIFICATION_DELIVERIES

          |
          v

     FCM/APNs

          |
          v

      Mobile App



A user can have multiple devices.

Therefore:

USERS 1 ---- N USER_DEVICES


------------------------------------------------------------------------

# 27. In-App Notification Flow**


Application

     |
     v

Notification Service

     |
     v

NOTIFICATIONS

     |
     v

IN_APP DELIVERY

     |
     v

Frontend

     |
     v

User reads notification



The notification can remain in the database until it is marked as read.

------------------------------------------------------------------------

# 28. WhatsApp Notification Flow**


Notification Service

       |
       v

WhatsApp Delivery

       |
       v

WhatsApp Provider

       |
       v

WhatsApp Platform

       | 
       v

     User



Provider callback:


SENT

  ↓

DELIVERED

  ↓

READ


------------------------------------------------------------------------

# 29. Voice Notification Flow


Notification Service

       |
       v

VOICE DELIVERY

       |
       v

Voice Provider

       |
       v

User Phone


Voice notifications should generally be reserved for important/critical events.

------------------------------------------------------------------------

# 30. Multi-Channel Notification**

A major advantage of this ERD is support for multiple channels.

Example:


Payment Failed

      |
      v

Notification #500

      |

      +---- EMAIL

      |

      +---- SMS

      |

      +---- PUSH

      |

      +---- IN_APP



Each delivery has its own status.

EMAIL   -> DELIVERED

SMS     -> DELIVERED

PUSH    -> FAILED

IN_APP  -> DELIVERED



------------------------------------------------------------------------

# 31. Priority

Recommended priority values:

text

LOW

NORMAL

HIGH

CRITICAL



Example:

Marketing       -> LOW

Order Update    -> NORMAL

Payment Alert   -> HIGH

Fraud Alert     -> CRITICAL



Priority can affect:

-   Queue selection

-   Retry policy

-   Provider selection

-   Quiet-hour bypass

-   Processing order

------------------------------------------------------------------------

# 32. Scheduling

For scheduled notifications:


NOTIFICATIONS

    |

    +-- scheduled_at


Example:


Create notification

       |
       v

scheduled_at = 2026-12-25 09:00

       |
       v

  Scheduler

       |
       v

     Queue

       |
       v

   Delivery



------------------------------------------------------------------------

# 33. Notification Status Model

              ┌───────────┐

              │  PENDING  │

              └─────┬─────┘

                    ↓

              ┌───────────┐

              │  QUEUED   │

              └─────┬─────┘

                    ↓

              ┌────────────┐

              │ PROCESSING │

              └─────┬──────┘

                    |

             ┌──────┴──────┐

             ↓             ↓

           SUCCESS       FAILURE

             ↓             ↓

        ┌──────────┐   ┌──────────┐

        │   SENT   │   │ RETRYING │

        └────┬─────┘   └────┬─────┘

             ↓              |

        DELIVERED           |

             ↓              ↓

           READ        Retry Limit

                            |

                            ↓

                         FAILED

                            |

                            ↓

                      DEAD LETTER



------------------------------------------------------------------------

# 34. Important Cardinalities

  Relationship                        Cardinality

  ----------------------------------- -------------

  User → Notifications                1:N

  User → Devices                      1:N

  User → Preferences                  1:1

  User → Channel Preferences          1:N

  User → Quiet Hours                  1:N

  Category → Notification Types       1:N

  Notification Type → Notifications   1:N

  Template → Notifications            1:N

  Notification → Recipients           1:N

  Notification → Deliveries           1:N

  Channel → Deliveries                1:N

  Provider → Deliveries               1:N

  Device → Deliveries                 1:N

  Delivery → Retry Attempts           1:N

  Delivery → Webhooks                 1:N

  Notification → Events               1:N

  Notification → Attachments          1:N

  Campaign → Notifications            1:N

------------------------------------------------------------------------

# 35. Recommended SQL Indexes

sql

CREATE INDEX idx_notifications_user_id

ON notifications(user_id);

CREATE INDEX idx_notifications_status

ON notifications(status);

CREATE INDEX idx_notifications_scheduled_at

ON notifications(scheduled_at);

CREATE INDEX idx_notifications_created_at

ON notifications(created_at);

CREATE INDEX idx_notifications_type_id

ON notifications(type_id);

CREATE INDEX idx_notifications_correlation_id

ON notifications(correlation_id);

CREATE INDEX idx_deliveries_notification_id

ON notification_deliveries(notification_id);

CREATE INDEX idx_deliveries_status

ON notification_deliveries(status);

CREATE INDEX idx_deliveries_provider_id

ON notification_deliveries(provider_id);

CREATE INDEX idx_retry_next_retry_at

ON notification_retry_attempts(next_retry_at);

CREATE INDEX idx_webhooks_provider_event_id

ON notification_webhooks(provider_event_id);

CREATE INDEX idx_devices_user_id

ON user_devices(user_id);

CREATE INDEX idx_devices_token

ON user_devices(device_token);


------------------------------------------------------------------------

## Additional Recommended Constraints

 sql

ALTER TABLE user_channel_preferences
ADD CONSTRAINT uq_user_channel_preferences
UNIQUE (user_id, channel_id);

ALTER TABLE notification_templates
ADD CONSTRAINT uq_notification_template_version
UNIQUE (template_key, channel_id, language, version);


These constraints prevent duplicate channel preferences for the same
user and allow multiple template versions without making `template_key`
itself unique.

------------------------------------------------------------------------

# 36. Why Separate Notification and Delivery?

Bad design:

text

NOTIFICATION

id

user_id

type

email_status

sms_status

push_status

whatsapp_status



This becomes difficult when new channels are added.

Better design:


NOTIFICATIONS

       |

       +---- NOTIFICATION_DELIVERIES

                 |

                 +---- EMAIL

                 +---- SMS

                 +---- PUSH

                 +---- WHATSAPP

                 +---- VOICE


Advantages:

-   Supports unlimited channels

-   Tracks each delivery independently

-   Easier retries

-   Easier provider fallback

-   Better normalization

-   Easier analytics

-   Easier maintenance

------------------------------------------------------------------------

# 37. Provider Fallback

A channel can have multiple providers.

Example:


SMS

 
 |

 +--> Provider A

 |

 X failure

 |

 +--> Provider B

 |
 v

Success


This is why `NOTIFICATION_PROVIDERS\` is separated from `NOTIFICATION_CHANNELS`.

------------------------------------------------------------------------

# 38. Example End-to-End Scenario

## Payment Failure
A payment service publishes:

json

{

  "event": "PAYMENT_FAILED",

  "userId": 101,

  "paymentId": "PAY123",

  "amount": 2500

}


Notification Service:

 text

PAYMENT_FAILED

       |
       v

Find user

       |
       v

Check preferences

       |
       v

Create notification

       |

       +---- EMAIL delivery

       |

       +---- SMS delivery

       |

       +---- PUSH delivery

       |

       +---- IN-APP delivery


Possible result:

text

EMAIL    -> DELIVERED

SMS      -> DELIVERED

PUSH     -> DELIVERED

IN-APP   -> READ



------------------------------------------------------------------------

# 39. Example Failure Scenario

 text

SMS Delivery

     |

     v

Provider Timeout

     |

     v

Retry Attempt #1

     |

     v

Provider Timeout

     |

     v

Retry Attempt #2

     |

     v

Provider Timeout

     |

     v

Retry Attempt #3

     |

     v

Move to Dead Letter


This gives operations teams a complete history.

------------------------------------------------------------------------

# 40. Normalization

The ERD follows relational database normalization principles.

## First Normal Form

Columns contain atomic values.

## Second Normal Form

Non-key attributes depend on the complete primary key.

## Third Normal Form

Independent concepts are separated into their own tables:


Users

Channels

Providers

Templates

Preferences

Notifications

Deliveries

Retries



This reduces duplication and update anomalies.

------------------------------------------------------------------------

# 41. High-Level Architecture

The ERD works with an event-driven architecture:


                 ┌──────────────────┐

                 │   Client / App   │

                 └────────┬─────────┘

                          |

                          v

                 ┌──────────────────┐

                 │   API Gateway    │

                 └────────┬─────────┘

                          |

                          v

                 ┌──────────────────┐

                 │ Notification API │

                 └────────┬─────────┘

                          |

              ┌───────────┼───────────┐

              v           v           v

         PostgreSQL     Redis       Kafka

                                      |

                                      v

                           Notification Worker

                                      |

             ┌────────────────────────┼─────────────────────┐

             v                        v                     v

           Email                      SMS                  Push

             |                        |                     |

         Provider                 Provider              FCM/APNs

             |                        |                     |

             └─────────────── Webhooks / Events ────────────┘



------------------------------------------------------------------------

# 42. Suggested Technology Stack


Backend:

Java 17+

Spring Boot

Spring Web

Spring Data JPA

Spring Security

Database:

PostgreSQL

Messaging:

Apache Kafka

Caching:

Redis

Authentication:

JWT

Testing:

JUnit 5

Mockito

MockMvc

Documentation:

OpenAPI / Swagger

Deployment:

Docker

AWS


------------------------------------------------------------------------

# 43. Explanation

A concise explanation:

> I designed the notification service around a logical notification and separate delivery records.

>  A notification represents the business event, while each delivery represents a channel-specific attempt such  as Email, SMS, Push, WhatsApp, or In-App. 

> This allows one notification to use multiple channels and maintain independent delivery statuses.

> separated channels from providers so that a channel can have multiple providers and support fallback.

> I also modeled user preferences, devices, templates, retries, webhooks, scheduling, campaigns, and dead-letter handling so the design can support both transactional and bulk notifications.

------------------------------------------------------------------------

# 44. Final ERD Summary


                          USERS

                            |

       ┌────────────────────┼────────────────────┐

       |                    |                    |
       v                    v                    v

   DEVICES             PREFERENCES          NOTIFICATIONS

                                                |
                     ┌──────────────────────────┼─────────────────────┐
                     |                          |                     |
                     |
                     v                          v                     v

                 ATTACHMENTS                 RECIPIENTS            DELIVERIES          

                                                |

                              ┌─────────────────┼─────────────────┐
                              |                 |                  | 
                              v                 v                 v

                          CHANNELS          PROVIDERS          DEVICES

                              |
                              v

                         DELIVERY

                              |

                 ┌────────────┼─────────────┐

                 v            v             v

              RETRIES      WEBHOOKS      EVENTS


The resulting design supports a complete notification lifecycle:


Business Event
      ↓

Notification

      ↓
Recipient Selection

      ↓
Preference Check

      ↓
Channel Selection

      ↓
Provider Selection

      ↓
Delivery

      ↓
Webhook / Status Update

      ↓
Retry if Required

      ↓
Delivered / Failed

      ↓
Audit History

------------------------------------------------------------------------

# 45. Design Notes and Production Rules

The ERD uses `NOTIFICATION_RECIPIENTS` as the link between users and a
logical notification. `NOTIFICATIONS` does not store a direct `user_id`,
because one notification may have many recipients.

`NOTIFICATION_DELIVERIES` stores the actual channel-level delivery for a
recipient. This means the same recipient can have multiple deliveries,
for example Email, SMS, Push, or WhatsApp.

For templates, `channel_id` points to `NOTIFICATION_CHANNELS`. Template
versions should be unique by the combination of template key, channel,
language, and version.

Recommended database constraint:

sql

UNIQUE (template_key, channel_id, language, version)


For user channel preferences, a practical constraint is:

sql
UNIQUE (user_id, channel_id)


Provider credentials should not be stored as plain text in the database.
`configuration_key` should point to a secret/configuration manager such
as AWS Secrets Manager, AWS Parameter Store, or another secure secret
store.

`NOTIFICATION_WEBHOOKS` represents callbacks received from external
providers. `NOTIFICATION_EVENTS` represents internal notification or
delivery state changes used for audit/history.

For a Kafka-based architecture, the operational dead-letter mechanism
should normally be a Kafka dead-letter topic. The
`DEAD_LETTER_NOTIFICATIONS` table can still be retained for
investigation, audit, and reprocessing metadata.

For multi-channel notifications, the overall `NOTIFICATIONS.status`
should be calculated from its deliveries. For example:

text
All deliveries successful       -> DELIVERED
Some successful, some failed    -> PARTIALLY_DELIVERED
All deliveries failed           -> FAILED
No delivery processed yet       -> QUEUED / PROCESSING


For a production implementation, the most important indexes are on
notification status and scheduled time, recipient/user lookup, delivery
status, retry time, provider event ID, and device/user lookup.

------------------------------------------------------------------------
------------------------------------------------------------------------

# 47. Conclusion

This ERD provides a flexible foundation for a production-grade

Notification Service.

It supports:

-   Email

-   SMS

-   Push

-   In-App

-   Web Push

-   WhatsApp

-   Voice

-   Multi-channel delivery

-   Multiple recipients

-   Templates

-   User preferences

-   Devices

-   Scheduling

-   Campaigns

-   Provider fallback

-   Retry handling

-   Webhooks

-   Delivery tracking

-   Audit history

-   Dead-letter processing

The key architectural decision is separating:


NOTIFICATION

      ↓
DELIVERY

      ↓
RETRY ATTEMPT


This separation makes the system extensible when new channels or providers are introduced.
