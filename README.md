# 🔔 Notification Service System – Database & System Design

<p align="center">
  <img src="./Notification_Service_Complete_ERD.png.png" alt="Notification Service Complete ERD" width="100%">
</p>

<h3 align="center">
Scalable Multi-Channel Notification Service
</h3>

---

## 📌 Project Overview

This project presents the database and system design of a scalable **Notification Service** capable of sending notifications through multiple communication channels.

The system is designed to support:

- Email
- SMS
- Push Notifications
- In-App Notifications
- Web Push
- WhatsApp
- Voice Notifications
- Multiple recipients
- Multiple notification channels
- Notification templates
- Scheduled notifications
- User notification preferences
- Provider management
- Provider fallback
- Retry mechanisms
- Delivery tracking
- Webhook callbacks
- Notification events and audit history
- Campaign-based notifications
- Dead-letter handling
- Attachments

The design focuses on **scalability, reliability, maintainability, extensibility, and fault tolerance**.

---

# 📑 Table of Contents

- [Project Overview](#-project-overview)
- [Problem Statement](#-problem-statement)
- [Objectives](#-objectives)
- [Key Features](#-key-features)
- [Complete ERD](#-complete-erd)
- [High-Level Architecture](#-high-level-architecture)
- [Notification Flow](#-notification-flow)
- [Database Design](#-database-design)
- [Table Descriptions](#-table-descriptions)
- [Entity Relationships](#-entity-relationships)
- [Multi-Channel Notification](#-multi-channel-notification)
- [Delivery Tracking](#-delivery-tracking)
- [Retry Mechanism](#-retry-mechanism)
- [Provider Management and Fallback](#-provider-management-and-fallback)
- [Webhook Handling](#-webhook-handling)
- [User Preferences](#-user-preferences)
- [Scheduled Notifications](#-scheduled-notifications)
- [Campaign Notifications](#-campaign-notifications)
- [Dead Letter Handling](#-dead-letter-handling)
- [Attachments](#-attachments)
- [Indexes and Performance](#-indexes-and-performance)
- [Data Integrity](#-data-integrity)
- [Security Considerations](#-security-considerations)
- [Scalability](#-scalability)
- [Reliability and Fault Tolerance](#-reliability-and-fault-tolerance)
- [Design Decisions](#-design-decisions)
- [Future Enhancements](#-future-enhancements)
- [Repository Structure](#-repository-structure)
- [Interview Explanation](#-interview-explanation)
- [Conclusion](#-conclusion)

---

# 🎯 Problem Statement

Modern applications need to communicate with users through different channels.

For example:

- An e-commerce application may send order confirmation through Email and SMS.
- A banking application may send security alerts through SMS, Email and Push.
- A food delivery application may send order updates through Push and WhatsApp.
- A SaaS application may send system alerts through Email and In-App notifications.

Building separate notification logic inside every application creates problems such as:

- Duplicate implementation
- Difficult provider management
- Poor scalability
- Difficult retry handling
- No centralized delivery tracking
- Difficult failure management
- Inconsistent notification templates
- Difficult preference management

Therefore, a centralized **Notification Service** can provide a common solution for all applications.

---

# 🎯 Objectives

The main objectives of this design are:

1. Support multiple notification channels.
2. Support multiple recipients.
3. Track every notification independently.
4. Track individual delivery attempts.
5. Support multiple external providers.
6. Implement provider fallback.
7. Support retries for failed deliveries.
8. Store notification templates.
9. Support scheduled notifications.
10. Support user notification preferences.
11. Handle provider webhooks.
12. Maintain notification history and audit events.
13. Support campaigns and bulk notifications.
14. Provide a scalable database structure.
15. Keep the design extensible for future channels.

---

# ⭐ Key Features

## 1. Multi-Channel Support

The service can support:

- Email
- SMS
- Push
- In-App
- Web Push
- WhatsApp
- Voice

New channels can be added without changing the core notification model.

---

## 2. Multiple Recipients

One notification can be sent to:

- One user
- Multiple users
- Multiple email addresses
- Multiple phone numbers
- Multiple devices

Recipients are stored separately in:

`NOTIFICATION_RECIPIENTS`

---

## 3. Multiple Deliveries

A single notification can have multiple delivery records.

For example:

```text
Notification
     |
     +---- Email Delivery
     |
     +---- SMS Delivery
     |
     +---- Push Delivery
