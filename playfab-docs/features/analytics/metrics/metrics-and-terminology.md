---
title: Metrics and Terminology
author: v-thopra
description: Contains tables with definitions for analytics terminology, engagement metrics, and revenue metrics.
ms.author: v-thopra
ms.date: 29/10/2018
ms.topic: article
ms.prod: playfab
keywords: playfab, analytics, metrics
ms.localizationpriority: medium
---

# Metrics and Terminology

## Engagement Metrics

| Metric                  | Definition                                                                                             |
| :-----------------------| :------------------------------------------------------------------------------------------------------|
| API Usage               | API usage statistics summarized by counts of **total calls**, **total successful calls**, and **total errors**.    |
| DAU                     | Daily active users. The count of unique players who logged in on a given day, from 12AM to 11PM UTC.   |
| Installs                | The count of installations on new devices, measured by the number of unique device IDs. This number may include both new and returning players.        |
| Logins                  | The count of player login events. **Unique Logins** counts the number of unique players. **Total Logins** counts all login events, which can be used to track number of sessions played. **Average Logins per User** tracks the average number of login events per active player.  Note: To obtain an accurate distribution of session counts, we recommend logging in the player upon each session start, even if the old login ticket is still valid.   |
| MAU                     | Monthly active users. In **Rolling Thirty Day Totals Report**, MAU refers to the count of unique players who logged in during the span of last 30 days. In **Monthly Totals Report**, MAU is calculated per month based on the first and last days of a given month.             |
| New Users               | The count of players who logged in for the first time, thus creating new player profiles.              |
| Retention               | The percentage of players who logged in again exactly N days after a reference date, marked as Day 0. **Thirty Day New User Retention Report** considers new users who joined the game within the span of last 30 days. **Thirty Day Retention Report** includes both new and returning users who logged into the game within the span of last 30 days.   |

## Revenue Metrics

| Metric                  | Definition                                                                                             |
| :-----------------------| :------------------------------------------------------------------------------------------------------|
| ARPDAU                  | Average revenue per daily active user in USD.
| ARPPU                   | Average revenue per paying user within a specified time period (e.g., hourly, daily, or monthly) in USD.
| ARPU                    | Average revenue per active user within a specified time period (e.g., hourly, daily, or monthly) in USD.
| Average Purchase Price  | The average transaction amount in USD (= Revenue/Purchases).
| Conversion              | The percentage of players who become paying users. **Thirty Day New User Conversion Report** tracks purchases made by new user cohorts over a 30-day period. Unlike retention reports, the conversion report tracks a cumulative total of players who converted within a given time span.
| Purchases               | The count of transactions.
| Top Items               | The top 100 items which generated the most revenue in your game within a specified report period.
| Top Spenders            | The top 100 players who spent the most money in your game within a specified report period.
| Transaction Revenue     | The sum of transaction amounts in USD. Transactions made in other currencies are converted into USD based on daily exchange rates.
| Unique Paying Users     | The count of unique players who spent money in your game.

## Terminology

| Metric               | Definition                                                                                             |
| :--------------------| :------------------------------------------------------------------------------------------------------|
| Cohort               | A group of players who joined or played the game on a given day, marked as Day 0.
| Event                | Events mark the transition between important states in your game, such as a player logging in, making a purchase, leveling up, or dying. A data event is a data structure that captures information about the event. PlayFab uses JSON to represent data events. **Standard events** are generated automatically during use of PlayFab services; games can generate **custom events** to track other important game-specific events. All data events can be saved, aggregated, reacted to, and analyzed via PlayFab’s real-time data pipeline, known as **PlayStream**.
| Platform             | PlayFab tracks four different types of platforms: **OS** (iOS, Android), **Device Type** (iPhone, iPad, Samsung Android, PC, Xbox, etc.), **Account Type** (Steam, Facebook, Kongregate, custom, etc.), and **Payment Type** (iTunes, Google Play, Steam, Facebook, Xsolla, etc.).
| Segment              | A group of players who share similar properties, e.g., geography, demographics, platform, purchase patterns, etc. In addition to **standard** segments, users can define **custom** segments. Note that segment definitions are limited to properties in the entity profile.
| Timestamp (TS)       | All PlayFab real-time analytics and reports are based on Coordinated Universal Time (UTC).

