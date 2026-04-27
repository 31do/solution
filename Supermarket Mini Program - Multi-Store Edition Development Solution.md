# Supermarket Mini Program - Multi-Store Edition Development Solution

**Huanqi Network | April 2026**

---

## I. Identified Pain Points

Chain supermarkets with multiple locations commonly face the following core challenges during digital transformation:

- **Difficult customer acquisition**: Over‑reliance on offline foot traffic and paper flyers, lack of online precision marketing and referral mechanisms – new customer acquisition remains costly and unsustainable.
- **Loose store management**: Each branch manages its own products, inventory, orders, and delivery operations. Headquarters cannot monitor real‑time performance across stores or allocate resources efficiently, leading to overstock in some locations and shortages in others.
- **Low customer loyalty**: Traditional membership cards are rarely used, purchase behaviors are not recorded, and personalised marketing is difficult. High churn rate – repeat purchase rate below 30%.
- **Inefficient delivery**: Most supermarkets still depend on third‑party food delivery platforms, which take 15‑20% commission, and brands have no control over delivery timeliness or service quality. Common complaints: "wrong store delivery" and "delivery timeout".
- **Weak technical capability**: Building an in‑house online system requires significant investment and long cycles. Generic SaaS platforms are function‑bound and cannot satisfy complex chain‑supermarket scenarios such as "nearest store assignment", "store‑independent inventory", and "scheduled delivery".

## II. Solution

Build a tailored "multi‑store O2O retail system" based on WeChat Mini Program, delivering the following core capabilities:

- **Integrated online & offline**: Consumers can browse products, place orders, and pay via the mini program; orders are automatically routed to the nearest deliverable store. Supports in‑store pickup and scheduled delivery.
- **Independent store operation**: Each branch has its own back‑end to manage products, inventory, orders, and delivery staff. Headquarters views data across all stores and centrally configures marketing campaigns.
- **Smart membership marketing**: Integrated tiered membership, points mall, coupons, flash sales, group buying, and order‑based discounts. The system automatically analyses purchase behaviour and pushes personalised offers.
- **Own delivery management**: Provides manager assignment or intelligent dispatch. The delivery staff app includes route navigation, photo proof of delivery, and historical track retention – full control over delivery cost and service quality.
- **Data‑driven decisions**: Real‑time reports on store sales, best‑selling products, repeat purchase analysis, providing accurate basis for replenishment, product selection, and promotions.

## III. Business Requirements

1. **Mini Program (User End)**: Support WeChat authorised login, multi‑level product categories, search, shopping cart, online payment, order tracking, member centre, point redemption, and participation in flash sales, group buying, coupons, etc.
2. **Management Backend (PC web)**:
   - Super admin: store management, global product management, campaign creation, data dashboard, permission assignment.
   - Store manager: own store’s product listing/unlisting, inventory adjustment, order handling, dispatch assignment, store‑specific report.
   - Delivery staff: receive dispatch, scan order QR code, start delivery, complete delivery (with photo).
3. **Multi‑store intelligent assignment engine**: Automatically assign the nearest deliverable store within 3 km based on user LBS; if outside all delivery ranges, guide to in‑store pickup.
4. **Data security & compliance**: Full‑chain SSL encryption, support for mini program filing agency service, daily automatic backup.
5. **High extensibility**: Smooth addition of fresh food delivery, wholesale marketplace, community group‑buy, etc.

## IV. Application Scenarios

| ID  | Scenario Name                        | Detailed Description                                                                                         |
|-----|--------------------------------------|---------------------------------------------------------------------------------------------------------------|
| S01 | User orders online, delivery from nearest store | User opens mini program at home, location automatically matched to nearest store. Browses products, adds to cart, selects “delivery”, pays, order is routed to that store. |
| S02 | User picks up in store               | User places order during work, selects “pick up in store”. Picks up goods quickly after work using a pickup code.      |
| S03 | Store manager processes order and assigns delivery staff | Store backend receives new order, manager prints receipt and assigns an available delivery person. Delivery staff receives task on their app. |
| S04 | Delivery staff takes photo as proof of delivery | After arrival, delivery person takes a photo of the goods at the doorstep/locker as proof and uploads to system, avoiding disputes. |
| S05 | Headquarters launches group‑buy campaign | Super admin creates “2‑person group‑buy” flash sale. Each store uses its own inventory. Users share group link to bring in friends – viral acquisition. |
| S06 | Member redeems points for gifts      | User accumulates points through purchases, redeems items (e.g., tissue, drinks) in points mall. Can be delivered with next order or picked up in store. |

## V. Application Architecture

| Layer              | Technology Components / Services                                                                   | Description                                                                                        |
|--------------------|-----------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| **Client Layer**   | WeChat Mini Program (user end), H5 management backend (store manager / admin / delivery staff)      | User end based on Vue + uni-app; backend uses Element UI + Vue.js                                   |
| **Access Layer**   | Nginx + SSL certificate + WAF firewall                                                              | Load balancing, HTTPS encryption, anti‑DDoS/CC, integrated with **Wangdao Multi‑Role Security Engine** for dynamic access control |
| **Application Layer** | C# .NET Core WebAPI (server), Python (AI Agent)                                                 | Business logic, authentication, order processing, promotion engine; AI Agent for intelligent dispatch and recommendation |
| **Data Layer**     | PostgreSQL (primary), Redis (cache), Elasticsearch (optional for product search)                    | Transaction data, cached sessions, hot product stats; **Wangdao Data Agent** used for heterogeneous data sync and automated reporting |
| **Infrastructure** | Tencent Cloud / Alibaba Cloud (starting 2C4G), OSS object storage (product images), SMS / WeChat notification channels | Daily auto‑backup, 99.95% availability; supports independent on‑premises deployment              |

## VI. User‑End Functions & Sections (WeChat Mini Program)

### 6.1 Homepage & Global Navigation

**Application scenario**: When a user first enters the mini program, they need quick visibility of supermarket promotions and popular products, plus navigation to categories, cart, and account.

**Implementation analysis**: Homepage uses a componentised design. Backend can dynamically configure carousel banners, diamond zone icons (flash sale, group buying, points mall, etc.), and promotional showcases (festival specials). Bottom tab bar fixed: Home, Categories, Cart, My.

**Technology / method**: Frontend Vue dynamic rendering. Backend provides configuration API (`/api/home/config`). Redis caches homepage config to reduce DB queries. **Wangdao UI Framework WDUI** builds responsive grid layout for different screen sizes.

**Algorithm & data flow**:
- User enters homepage → requests config API → backend reads JSON config from Redis → if missing, reads from DB and writes to Redis → returns to frontend.
- Product recommendation: weighted score = `0.6 * last‑7‑day sales + 0.4 * user preference match` (preference based on last 5 browsed product IDs stored in Redis).

**Operation flow**: Pull down mini program → see carousel of weekly special fruits → click “Flash Sale” in diamond zone to enter activity page → switch via bottom tabs.

**FAQ**:
- Q: How often can carousel images be changed? A: Super admin can upload new images anytime, set display start/end time, effective immediately.
- Q: Slow homepage loading? A: CDN for images + Redis cache; normal load time <1 second.

### 6.2 Product Categories & Search

**Application scenario**: User wants to buy beverages, drills down from root category to “carbonated drinks”, or directly searches “Coke” and sorts by price or popularity.

**Implementation analysis**: Three‑level categories. Search supports fuzzy match and Pinyin initial letters (“kel” finds Coke). Results can be sorted by relevance, sales, price.

**Technology / method**: PostgreSQL stores category tree (parent_id recursive query). Search uses `ilike` full scan (acceptable for <50k products; can upgrade to Elasticsearch). Sorting via SQL order by.

**Algorithm & data flow**:
- Category page: request category ID → backend queries all products under that category (including sub‑categories via PostgreSQL recursive CTE) → paginated return.
- Search flow: user types keyword → frontend debounce 500ms → backend `name ilike '%keyword%'` → sorted.

**Operation flow**: Tap “Categories” bottom → see Food, Daily Necessities, Fresh Food → tap “Food” expands level‑2 → tap “Beverages” expands level‑3 “Carbonated / Juice” → product list.

**FAQ**:
- Q: Duplicate categories? A: Name uniqueness enforced, drag‑drop ordering supported.
- Q: Different category order per store? A: Yes. Backend can configure `store_category_order` table.

### 6.3 Product Detail Page

**Application scenario**: User taps a product, sees large images, price, stock, options (330ml/500ml), adds to cart or buys now.

**Implementation analysis**: Detail page shows the specific store’s selling price and stock (in multi‑store edition, price and stock can differ per store). Supports multi‑spec SKU (flavour, weight). Integrated **Wangdao Multi‑Dimensional SKU Engine** to handle tens of thousands of SKU combinations with real‑time inventory deduction.

**Technology / method**: SPU table, SKU table, store‑product relation table (`store_spu`) stores store‑specific price and stock. Adding to cart triggers Redis atomic pre‑reservation (prevents overselling). Skeleton screen improves perceived performance.

**Algorithm & data flow**:
- Get detail: based on `spu_id` and current store `store_id`, query `store_spu` for price & stock; query SKU table for spec list.
- Spec selection: user selects spec (e.g., “500ml”) → frontend shows corresponding SKU stock, price → adding to cart writes to Redis cart hash.

**Operation flow**: Tap product → view multiple images → choose “500ml” spec → tap “Add to Cart” → success prompt.

**FAQ**:
- Q: Prices differ per store, confusing? A: Mini program top bar shows current store name; user can manually switch stores, prices are clear.
- Q: Price drops after adding to cart? A: Checkout re‑fetches latest price, final price at order submission.

### 6.4 Shopping Cart & Checkout

**Application scenario**: User adds multiple products, adjusts quantities, deletes items, then selects store, delivery method, coupon, and places order.

**Implementation analysis**: Cart data stored in Redis (key = user ID, hash: field = sku_id+store_id, value = quantity). At checkout, backend re‑validates each product’s latest price, stock, and delivery range.

**Technology / method**: Redis cart TTL 30 days. Order submission uses DB transaction: decrease store stock (`update ... where stock >= quantity`), create order, clear corresponding cart items. Rollback on insufficient stock.

**Algorithm & data flow**:
- Checkout request: backend iterates cart items → queries store‑product current price, stock → calculates total → queries user’s usable coupons (unused, not expired, meeting threshold) → returns to frontend.
- Place order: frontend submits order request → backend starts transaction → locks stock row (`select for update`) → decreases → generates order ID (snowflake) → inserts order table → clears cart items → commits.

**Operation flow**: Cart list selects items → taps “Checkout” → selects delivery address (add if first time) → chooses “Immediate delivery” and schedules time slot → system automatically matches store → uses coupon → submits order.

**FAQ**:
- Q: Products from different stores in cart? A: System splits order by store. One payment generates multiple sub‑orders, each delivered separately.
- Q: Unpaid order – stock locked? A: Locked for 15 minutes. After timeout, automatic stock release via scheduled job.

### 6.5 Order & Payment

**Application scenario**: After submitting order, user goes to payment page and calls WeChat Pay. After success, order status can be tracked (pending delivery, delivering, completed, cancelled). User can request refund (only for undelivered orders).

**Implementation analysis**: Async callback updates order status after successful payment. Refunds support original channel return, requiring admin approval.

**Technology / method**: WeChat Pay V3 API, generate prepay order. Refund uses API certificate. Rigorous order state machine to prevent illegal state transitions.

**Algorithm & data flow**:
- Payment flow: frontend gets `prepay_id` → calls `wx.requestPayment` → success → WeChat async notifies `/pay/notify` → backend verifies signature → updates order status to “pending delivery” → sends template message.
- Refund flow: user applies → admin calls WeChat refund API → WeChat async notifies result → updates order status to “refunded”.

**Operation flow**: Submit order → WeChat Pay fingerprint/password → success → receives WeChat service notification → order list shows “pending delivery”.

**FAQ**:
- Q: Payment succeeds but order status not updated? A: Async notification may be delayed. System provides manual sync button (active query to WeChat Pay).
- Q: Refund arrival time? A: WeChat Pay original‑channel return usually 1‑3 business days.

### 6.6 Member Centre

**Application scenario**: User views membership tier, points, coupons, delivery addresses, order history, points mall redemption, etc.

**Implementation analysis**: Membership tier auto‑upgrades based on cumulative spending (e.g., V1:0‑499 yuan, 5% off; V2:500‑999, 10% off; V3:≥1000, 15% off). Points rule: 1 yuan spent = 1 point. Points can be used for product redemption or as cash (100 points = 1 yuan).

**Technology / method**: Member table has `level`, `total_amount`. On order completion, accumulate `total_amount` and re‑compute level (event triggered). Point ledger table.

**Algorithm & data flow**:
- Tier calculation: `order_complete` event → update `total_amount` → compare with tier thresholds → update `level` → send template message “Congratulations on upgrade”.
- Points discount: on checkout, user chooses “use points” → backend calculates max discount = min(points/100, order_amount*0.3) → records deduction in order table.

**Operation flow**: Tap “My” → shows V2, points balance 560 → tap “Points Mall” → redeem tissue (200 points) → confirm → gets redemption code.

**FAQ**:
- Q: Points from different stores combined? A: Yes, all purchases across stores contribute to same account.
- Q: Points expire? A: Backend can set expiry (e.g., end of calendar year), reminder 15 days before expiry.

### 6.7 Promotion Participation (Flash Sale / Group Buying / Coupons)

**Application scenario**: User sees flash sale countdown on homepage and participates; or initiates a group buy and shares to friends; or claims coupons.

**Implementation analysis**: Flash sale uses separate inventory isolated from regular stock. Group buy requires dedicated order table and group table. Coupons support claiming and distribution.

**Technology / method**: Flash sale pre‑warms inventory in Redis, atomic deduction via Lua script. Group buy uses state machine, auto‑refund if group fails within time limit. Coupon distribution supports “new user registration”, “spend to get coupon”, etc.

**Algorithm & data flow**:
- Flash sale order: user request → Redis Lua decrement flash sale stock → if success, async create order (decrease product flash stock) → else return “sold out”.
- Group buy: user starts group → creates group record (24h validity) → payment success → waits for others to join → after joining, check if group is full → if full, all member orders change to “pending delivery”; if timeout, auto refund.

**Operation flow** (group buy): User A taps “2‑person group buy” → pays 39.9 yuan → shares to WeChat friend B → B enters group and pays → both get product.

**FAQ**:
- Q: Can flash sale products be combined with regular products in one order? A: No, flash sale items must be ordered separately to simplify inventory logic.
- Q: Group buy fails – refund deducts coupon? A: Refund returns the coupon (if unused) or partially according to ratio.

## VII. Backend Functions (PC Management End)

### 7.1 Store Management (Super Admin)

**Application scenario**: Headquarters creates branches (East City Store, South City Store), sets each store’s delivery radius, business hours, and store manager accounts.

**Implementation analysis**: `store` table contains name, address, lat/lng, delivery radius (m), opening/ closing time, status (enabled/disabled). Admin accounts linked to `store_id`.

**Technology / method**: PostGIS for lat/lng and distance calculation. Backend map picker (Tencent Maps API) obtains coordinates. **Wangdao Multi‑Role Security Engine** provides RBAC granularity to prevent privilege escalation.

**Data flow**: Super admin adds a store → writes to `stores` table → creates independent product management authority for that store.

**Operation flow**: Super admin logs into backend → clicks “Store Management” → New store → fills info, marks location on map → Save.

**FAQ**:
- Q: Can a store have multiple store managers? A: Yes, assign multiple admin accounts to the same store ID.
- Q: Changing delivery radius affects existing orders? A: No, only new orders use the new radius.

### 7.2 Product Management (Store Manager + Super Admin)

**Application scenario**: Store manager manages own store’s products: list/unlist, adjust price and stock, set flash sale stock, batch import (barcode scanner).

**Implementation analysis**: Supports single entry and batch import. When scanning a barcode, system searches central product base. If found, auto‑fills name, image, category; if not found, pops up a new form. Stock alert: highlights when stock below threshold.

**Technology / method**: Central product base table (`product_base`) stores standard info. Store‑product table (`store_product`) stores store‑specific price, stock, listing status. Barcode scanner input simulates keyboard event; frontend listens for `Enter` key to trigger lookup. **Wangdao Data Agent** automatically syncs product base changes to all stores.

**Algorithm & data flow**:
- Fast barcode entry: scan barcode → backend queries `product_base` → if exists, returns product info; store manager only inputs store price & stock → writes to `store_product`; if not exists, enters new base info flow.
- Stock alert: scheduled job (hourly) scans `store_product.stock < alert_threshold` → sends in‑app notification to store manager.

**Operation flow**: Store manager logs into backend → Product Management → “Add Product” → scans barcode with scanner → system auto‑fills name, category → inputs store price 99 yuan, stock 100 → Save.

**FAQ**:
- Q: Headquarters adjusts product price, how to sync to all stores? A: Super admin modifies “suggested retail price” in base product; each store can manually click “Sync price”.
- Q: Recommended product image size? A: 750px width, 1:1 ratio, system compresses automatically.

### 7.3 Order Processing & Delivery Dispatch

**Application scenario**: Store receives new order, manager views details, prints receipt, dispatches to a delivery person (or enables auto‑dispatch). Delivery staff receives task, updates delivery status.

**Implementation analysis**: Order list grouped by status (pending delivery, delivering, completed). Dispatch supports manual assignment and intelligent recommendation based on distance and idle status. Delivery staff uses standalone mini program or H5.

**Technology / method**: WebSocket pushes new order notification. Intelligent dispatch uses greedy algorithm: score = `(distance_km * 2) + (current_orders * 5)`. Upon delivery completion, staff takes photo and uploads to OSS.

**Algorithm & data flow**:
- Intelligent dispatch: new order appears → queries online delivery staff of that store → computes score per staff (`distance_km*2 + current_orders*5`) → selects the lowest score → pushes order.
- Delivery track: after tapping “Start Delivery”, GPS reported every 30 seconds, stored in location history table (for dispatch monitoring).

**Operation flow**: Store manager logs into backend → order list shows new order (red badge) → clicks details → prints receipt → clicks “Assign delivery staff” → selects Xiao Zhang → Xiao Zhang receives task on delivery app → scans order QR code → picks goods → starts delivery → takes photo on arrival → order status becomes “Completed”.

**FAQ**:
- Q: What if delivery staff rejects the order? A: Manager can reassign or mark “no one available” and SMS user to pick up.
- Q: How to prevent fake delivery proof? A: Require photo + geolocation check (GPS must be within 100m of delivery address when marking completed).

### 7.4 Marketing Campaign Management (Super Admin)

**Application scenario**: Headquarters creates coupons, flash sales, group buying, order‑based discounts, and point‑redemption campaigns.

**Implementation analysis**: Campaigns can be applied to all stores or a subset. Flash sales and group buying require activity inventory and per‑user purchase limits. Coupons support rules like “spend 100, get 10 off”.

**Technology / method**: Activity configuration table stores rules in JSON. Scheduled tasks pre‑warm flash sale inventory into Redis before activity starts. Group‑buy timeout uses delayed queue (RabbitMQ or Redis key‑expiry listener).

**Algorithm & data flow**:
- Flash sale start: admin sets start/end time → 30 minutes before start, load inventory into Redis key `seckill:{activity_id}:{sku_id}` → on purchase, Lua atomic decrement → after activity ends, key deleted.
- Auto‑match order‑based discount: at checkout, backend iterates enabled discounts, computes best combination (simple linear match due to limited store‑level campaigns).

**Operation flow**: Admin logs into backend → Marketing Centre → New Flash Sale → selects “Imported Cherries”, store price 9.9 yuan, activity stock 50, limit 1 per user → sets time 20:00‑20:30 → Publish → mini program homepage shows countdown.

**FAQ**:
- Q: Can flash sale be combined with other discounts? A: No, flash sale products are excluded from other order‑based discounts and coupons to prevent excessive loss.
- Q: Group buy fails – refund fees? A: WeChat Pay refunds full amount; merchant does not bear any fee (original transaction fee is returned).

### 7.5 Data Dashboard & Reports

**Application scenario**: Headquarters views sales trends across all stores, top products, user analysis. Store manager views store‑specific reports.

**Implementation analysis**: Dashboard shows yesterday’s sales, month‑over‑month growth, real‑time today’s orders. Charts using ECharts. Supports export to Excel.

**Technology / method**: Offline statistics: daily 2 AM scheduled task aggregates order data (`order_detail` + `order`) into report summary tables. Dashboard queries summary tables to avoid real‑time aggregation pressure. Export using NPOI component. **Wangdao Data Agent** automatically generates weekly reports and emails to store managers.

**Algorithm & data flow**:
- Sales MoM: (today – yesterday) / yesterday.
- Best‑selling products: `order_detail` table `group by sku_id`, sum(quantity) for last 7 days, top 10.

**Operation flow**: Super admin enters “Data Dashboard” → sees bar chart of store sales → clicks “Export Monthly Report” → downloads Excel.

**FAQ**:
- Q: Real‑time dashboard? A: Today’s data is real‑time (query order table); historical data from summary tables, error within 5 minutes.
- Q: Can we see repurchase rate per user? A: Yes, “User Analysis” module provides repurchase funnel (first purchase → second purchase ratio).

### 7.6 Permission & Security Management

**Application scenario**: Super admin creates roles (operations, finance), assigns menu permissions; sets IP whitelist, optional two‑factor authentication.

**Implementation analysis**: RBAC model: user → role → permission. Granular to button level (e.g., “Export reports”). Security: integrated **Wangdao Encryption Engine** for database‑level encryption of sensitive data (mobile phone, address).

**Technology / method**: JWT token authentication + refresh token. Encryption uses AES‑256. Login logs record IP, device.

**Operation flow**: Super admin → Role Management → New “Finance Role” → ticks “View reports” permission → assigns finance staff account to that role.

**FAQ**:
- Q: Store manager accidentally deletes a product? A: System has recycle bin; super admin can restore.
- Q: How to prevent employee data leakage? A: Backend can enable watermark (username + timestamp) on all exported Excel files.

## VIII. Security Strategy

- **Transmission security**: Full‑site SSL/TLS 1.3, mandatory HTTPS, man‑in‑the‑middle protection. SSL certificates provided and auto‑renewed by Huanqi as agency service.
- **Storage security**: User passwords bcrypt‑hashed with salt. Mobile numbers and addresses encrypted with AES‑256; encryption keys stored in a separate KMS. **Wangdao Encryption Engine** supports field‑level dynamic masking – backend operators see only first and last 3 digits.
- **Application security**: SQL injection prevention (parameterised queries), XSS filtering, CSRF token. Rate limiting (max 10 requests per second per IP). **Wangdao Multi‑Role Security Engine** provides fine‑grained access control and automatically blocks abnormal login behaviours (unusual location, brute force).
- **Backup & disaster recovery**: Daily full backup of database and images, retained for 30 days; one‑click restore. Cloud server 99.95% availability.
- **Filing & compliance**: Agency service for mini program filing and website filing, ensuring compliance with Personal Information Protection Law; privacy policy pop‑up for user consent.

## IX. Feature Combinations

| Combination Name        | Core Modules                                                                                                 | Advantages                                                                                       |
|-------------------------|--------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| **High Value for Money** | Basic user end (homepage, categories, cart, orders, member centre)<br>Basic backend (product management, order handling, single‑store report)<br>Payment & basic dispatch | Fast go‑to‑market, lowest cost. Suitable for single store or online trial.                      |
| **Optimal Combination** | High Value for Money + <br>Multi‑store independent inventory & permissions<br>Coupons, order‑based discounts<br>Dashboard (headquarters)<br>Auto dispatch + delivery photo proof | Full chain‑store management, marketing & delivery closed loop, best balance of function and cost. |
| **Flagship Combination** | Optimal Combination + <br>Flash sale, group buying, points mall<br>Smart recommendation engine (based on Wangdao Data Agent)<br>AI predictive replenishment (pilot)<br>Customised Business Algorithm Engine (GEO local search optimisation) | Comprehensive digital retail hub, intelligent marketing and decision‑making. Best for regional leading supermarkets. |

## X. Important Notes

1. **Data initialisation**: Before going live, complete product information, store configuration, and delivery staff registration. It is recommended to finish image and text entry for the first 100 high‑frequency products – Huanqi provides this free of charge.
2. **User privacy**: The mini program must have a privacy protection guide, clarifying the purpose of collecting location and mobile phone number. First‑time user must authorise.
3. **Payment integration**: Apply for WeChat Pay merchant account in advance, configure the callback domain. Ensure sufficient balance for refunds.
4. **Delivery range testing**: The system uses straight‑line distance for range calculation; actual walking route may differ. Suggest each store test and fine‑tune the radius.
5. **After‑sales rules**: Clearly state that fresh products are non‑returnable. For other products, refunds are allowed only before delivery. For delivered items, user must reject the goods and store verifies before refund.
6. **High concurrency**: For flash sale events, temporarily scale server (2C4G → 4C8G), then scale back after event ends.
7. **System maintenance**: Update SSL certificates at least monthly (Huanqi provides automatic reminders); regularly purge cache logs older than 30 days.

## XI. Extended Thinking

1. **ERP integration**: In the future, the supermarket’s existing ERP for purchase/sales/inventory can be integrated with the mini program orders and inventory to achieve automatic procurement and automatic stock deduction. Huanqi, based on the Wangdao Data Agent, has already accumulated various ERP interface adaptation solutions.
2. **Community group‑buy extension**: The multi‑store edition can be extended to a “group leader + pickup point” model, using existing stores as pickup points, adding pre‑sale, next‑day delivery functions.
3. **AI smart recommendation**: Train a LightGBM model based on users’ historical purchase behaviour to predict product preferences, enabling personalised “Recommended For You” on the homepage and increasing conversion rate.
4. **GEO local search optimisation**: Through the **Wangdao Business Algorithm Engine**, improve the mini program’s ranking for keywords like “nearby supermarket” in WeChat Search, driving natural traffic.
5. **Multi‑channel integration**: Later, add Douyin mini program and Alipay mini program, all managed by a unified backend, achieving omnichannel retail.

## XII. Terms & Definitions

| Term                               | Definition                                                                          |
|------------------------------------|-------------------------------------------------------------------------------------|
| **SPU**                            | Standard Product Unit, e.g., “Coca‑Cola” is one SPU.                                 |
| **SKU**                            | Stock Keeping Unit, e.g., “330ml*24 cans” is one SKU.                                |
| **O2O**                            | Online to Offline.                                                                  |
| **LBS**                            | Location Based Service.                                                              |
| **RBAC**                           | Role‑Based Access Control.                                                          |
| **Wangdao Multi‑Dimensional SKU Engine** | Huanqi’s proprietary high‑performance product specification processor, supporting unlimited cascading SKUs and real‑time inventory deduction. |
| **Wangdao Multi‑Role Security Engine** | Security middleware providing fine‑grained permission control, dynamic masking, and abnormal behaviour detection. |
| **GEO**                            | Local ranking optimisation for search engines, improving the mini program’s visibility in local search results. |

## XIII. References

1. WeChat Mini Program Development Documentation – WeChat Open Platform, 2025.
2. *Chain Supermarket Digital Transformation White Paper* – China Chain Store & Franchise Association, 2024.
3. Huanqi Network, *Wangdao Data Engine Technical Architecture V3.0* internal document.
4. PostgreSQL 15 Official Manual – PostgreSQL Global Development Group.
5. *Retail Industry Data Security Implementation Specification* GB/T 42018‑2023.

---

**Huanqi Network**  
Planner: Miss Shan 13215191218  
WeChat: wancome  
April 27, 2026  

Dongguan Huanqi Network Information Co., Ltd.  
Flagship brand: Wangdao  
20 years of technical expertise | 50+ IP rights | 300+ products | 160,000+ enterprise customers