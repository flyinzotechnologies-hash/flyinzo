# FLYINZO — FIREBASE SCHEMA (Build Reference v1)

> Exact field names and types. Codex must use these names verbatim across main app, admin app, and Cloud Functions. Never invent a field. If a screen needs data not listed here, stop and report it instead of adding a field silently.

## Collections Index
`users`, `userProfiles`, `adminUsers`, `deals` (+ `private/booking`, `private/returnPlan` subdocs), `airports`, `mediaAssets`, `appConfig`, `featureFlags`, `users/{uid}/entitlements/premium`, `userActivity`, `dailyUsage`, `dealAnalytics`, `dealReports`, `auditLogs`. Future schema-only: `merchProducts`, `orders`, `stopoverHubs`, `stopoverRules`, `reelsContent`, `notifications`.

---

### `users/{uid}`
```
uid: string
email: string
displayName: string
photoURL: string | null
authProvider: "google" | "email"
accountStatus: "active" | "banned" | "deleted"
userAccessSegment: "guest" | "free_normal" | "free_limited" | "free_high_intent" | "premium" | "admin" | "banned"
isPremium: boolean
premiumSource: "revenuecat" | "manual" | "promo" | null
subscriptionStatus: "none" | "active" | "expired" | "cancelled" | "trial" | "grace_period"
subscriptionPlan: "free" | "day_pass" | "monthly" | "yearly" | "manual" | "promo"
revenueCatCustomerId: string | null
premiumUntil: timestamp | null
homeAirportIata: string | null
homeCity: string | null
homeCountry: string | null
preferredCurrency: string
preferredLanguage: string
createdAt: timestamp
updatedAt: timestamp
lastActiveAt: timestamp
```
**Client may write:** displayName, photoURL, homeAirportIata, homeCity, homeCountry, preferredCurrency, preferredLanguage.
**Client may never write:** isPremium, premiumSource, subscriptionStatus, subscriptionPlan, userAccessSegment, accountStatus.

### `userProfiles/{uid}`
```
uid: string
fullName: string | null
phoneNumber: string | null
addressLine1: string | null
addressLine2: string | null
city: string | null
state: string | null
country: string | null
pinCode: string | null
savedAddresses: array
createdAt: timestamp
updatedAt: timestamp
```
Never required at signup. User edits own doc.

### `adminUsers/{uid}`
```
uid: string
email: string
displayName: string
role: "founder" | "super_admin" | "admin" | "manager" | "staff" | "intern" | "viewer"
permissions: array<string>
isActive: boolean
invitedBy: string | null
createdAt: timestamp
updatedAt: timestamp
lastLoginAt: timestamp
```
Only founder/super_admin may create or modify this collection (except a user's own `lastLoginAt`).

### `deals/{dealId}` — PUBLIC, safe for client read
```
dealId: string
title: string
notes: string | null
dealType: "direct" | "stopover" | "multi_city"
tripType: "one_way" | "round_trip" | "multi_city"
originIata: string
originCity: string
originCountry: string
originTz: string | null
destinationIata: string
destinationCity: string
destinationCountry: string
destinationTz: string | null
stopoverIata: string | null
stopoverCity: string | null
stopoverCountry: string | null
dateOptions: array
currencyCode: string
price: number
mrp: number | null
discountPercent: number | null
airlineCode: string | null
airlineName: string | null
flightNumber: string | null
stopCount: number | null
durationMinutes: number | null
flightLegs: array
layovers: array
totalFlightTimeMinutes: number | null
totalTripDurationMinutes: number | null
urgency: "low" | "medium" | "high"
dealKind: "regular" | "flashSale" | "limitedTime" | "mistakeFare"
isFeatured: boolean
isStarred: boolean
priorityRank: number | null
rank: number
adminRating: number
publicRatingEnabled: boolean
publicRatingAverage: number | null
publicRatingCount: number
visibility: "free" | "premium" | "premiumFirst"
premiumDelayHours: number | null
status: "draft" | "published" | "hidden" | "expired" | "archived"
launchAt: timestamp | null
expireAt: timestamp | null
lastCheckedAt: timestamp | null
shortDescription: string
teaserText: string | null
note: string | null
imageUrl: string | null
imageAssetId: string | null
imageProvider: string | null
createdAt: timestamp
updatedAt: timestamp
createdBy: string
updatedBy: string
```
**FORBIDDEN in this document, no exceptions:** `bookingUrl`, `affiliateUrl`, any provider redirect URL, private admin notes, full return-plan text.

### `deals/{dealId}/private/booking` — PROTECTED
```
bookingUrl: string
provider: string | null
affiliateCode: string | null
sourceLink: string | null
lastCheckedAt: timestamp | null
createdAt: timestamp
updatedAt: timestamp
updatedBy: string
```
Admin: read/write. Premium user: read only if deal entitlement check passes. Free/guest: no read access (guest blocked entirely if `bookingRequiresLogin` is true).

### `deals/{dealId}/private/returnPlan` — PROTECTED
```
hasReturnPlan: boolean
returnPlanTitle: string | null
returnPlanPreview: string | null
returnPlanFullText: string | null
returnBookingLink: string | null
returnRouteNotes: string | null
premiumOnly: boolean
createdAt: timestamp
updatedAt: timestamp
updatedBy: string
```
`returnPlanPreview` may be exposed publicly via the parent deal doc's teaser if admin allows; `returnPlanFullText` is premium-gated when `premiumOnly` is true.

### `airports/{iataCode}`
```
iataCode: string
icaoCode: string | null
name: string
city: string
cityLower: string
nameLower: string
countryCode: string
countryName: string
countryLower: string
latitude: number
longitude: number
timezone: string | null
isPriorityHub: boolean
```
Imported once from existing seed file. `countryLower` added during import if missing from source data. Public read allowed (needed for client-side search); write restricted to authorized admin.

### `mediaAssets/{assetId}`
```
assetId: string
imageUrl: string
storagePath: string | null
title: string
city: string | null
country: string | null
region: string | null
category: "city" | "country" | "beach" | "mountain" | "cityscape" | "airport" | "generic_travel" | "luxury" | "budget_travel"
tags: array<string>
provider: string | null
source: string | null
usedCount: number
createdAt: timestamp
updatedAt: timestamp
createdBy: string
```
Public read (images render in user app). Write restricted to admin/staff with image permission.

### `appConfig/global`
```
appName: string
maintenanceMode: boolean
guestBrowsingEnabled: boolean
guestDealLimit: number
bookingRequiresLogin: boolean
premiumCardsVisible: boolean
premiumBlurPercentage: number
freeDailyDealDetailLimit: number
freeDailyBookingClickLimit: number
supportEmail: string
supportWhatsappNumber: string | null
instagramUrl: string | null
privacyUrl: string | null
termsUrl: string | null
updatedAt: timestamp
updatedBy: string
```
Public read (client needs it at app entry). Write restricted to settings.edit permission.

### `featureFlags/{featureKey}`
```
key: string
status: "enabled" | "disabled" | "coming_soon" | "admin_only" | "premium_only"
title: string
description: string
updatedAt: timestamp
updatedBy: string
```
Keys: `deals, premiumDeals, guestBrowsing, pastDeals, stopovers, merchandise, reels, language, tripPlanner, smartScraper, nearbyAirports, notifications, ratings, banners`. Public read, restricted write.

### `users/{uid}/entitlements/premium`
```
active: boolean
plan: string
source: "revenuecat" | "manual" | "promo"
revenueCatCustomerId: string | null
productId: string | null
originalTransactionId: string | null
expiresAt: timestamp | null
updatedAt: timestamp
```
**Client read-only.** Write only via trusted Cloud Function (RevenueCat webhook) or authorized admin manual-grant action with `premium.grant` permission. Manual grants must also write an `auditLogs` entry.

### `userActivity/{eventId}`
```
eventId: string
eventType: string
uid: string | null
anonymousId: string | null
userAccessSegment: string
dealId: string | null
platform: "ios" | "android" | "web"
appVersion: string
sessionId: string
metadata: map
createdAt: timestamp
```
Client may create (restricted field validation), never edit/delete. Admin-only read.

### `dailyUsage/{uid_YYYYMMDD}`
```
uid: string
date: string
appOpenCount: number
sessionCount: number
dealCardViews: number
dealDetailOpens: number
bookingLinkClicks: number
premiumLockedClicks: number
upgradePromptShownCount: number
upgradeClicks: number
guestLimitReachedCount: number
freeLimitReachedCount: number
lastActiveAt: timestamp
```
Client cannot directly edit own summary in production (Phase 1 may allow restricted incremental writes; Cloud Function aggregation is the long-term path).

### `dealAnalytics/{dealId}`
```
dealId: string
cardViews: number
detailOpens: number
bookingClicks: number
premiumLockClicks: number
upgradeClicksFromDeal: number
reportsCount: number
expiredReportsCount: number
wrongPriceReportsCount: number
lastClickedAt: timestamp | null
lastViewedAt: timestamp | null
```
Admin read. Writes via Cloud Function or restricted client increment.

### `auditLogs/{logId}` — APPEND-ONLY
```
logId: string
action: string
adminUid: string
adminEmail: string
adminRole: string
targetType: string
targetId: string
before: map | null
after: map | null
metadata: map | null
createdAt: timestamp
```
Created automatically by every privileged admin write. Never editable or deletable from any client. Read restricted to `auditLogs.read` permission (founder/super_admin by default).

### `dealReports/{reportId}`
```
reportId: string
dealId: string
uid: string | null
anonymousId: string | null
reason: "expired" | "wrong_price" | "wrong_link" | "wrong_route" | "wrong_date" | "other"
message: string | null
status: "open" | "reviewed" | "resolved" | "ignored"
createdAt: timestamp
reviewedAt: timestamp | null
reviewedBy: string | null
```
Any authenticated or anonymous user may create. Admin manages status.

### Future schema-only collections (do not build screens/logic yet)
`merchProducts/{productId}`, `orders/{orderId}`, `stopoverHubs/{hubId}`, `stopoverRules/{ruleId}`, `reelsContent/{contentId}`, `notifications/{notificationId}` — field shapes documented in MASTER_PRD future-modules reference when needed; not required for Phase 1 build prompts.
