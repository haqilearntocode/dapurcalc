# DapurCalc V1 --- Product Requirements Document (PRD)

**Status:** Draft siap implementasi\
**Tujuan:** Portfolio MVP + uji penggunaan nyata\
**Platform utama:** Mobile-first responsive web application / PWA\
**Bahasa UI:** Indonesia\
**Target pengguna V1:** Satu Admin / Pemilik Usaha

------------------------------------------------------------------------

## 1. Product Overview

DapurCalc adalah aplikasi sederhana untuk membantu usaha makanan rumahan
mencatat pembelian bahan, mengelola stok, menghitung estimasi kebutuhan
produksi, mencatat penggunaan bahan aktual, menyimpan hasil produksi
sebagai stok produk jadi, memenuhi pesanan, dan menghitung biaya serta
laba secara lebih terukur.

V1 berfokus pada workflow usaha makanan rumahan dengan produk berbasis
**pcs**, menggunakan **Pempek Lenjer Sarden** sebagai skenario utama.

DapurCalc V1 bukan sistem akuntansi penuh dan bukan SaaS. V1 harus
kecil, dapat selesai, dapat digunakan melalui smartphone, dan dapat
diuji langsung oleh pengguna nyata.

------------------------------------------------------------------------

## 2. Problem Statement

Workflow saat ini memiliki beberapa masalah:

-   Pesanan terutama masuk melalui WhatsApp dan dicatat manual.
-   Harga bahan dapat berubah antar pembelian.
-   Pembelian bahan sering dianggap sebagai "modal", padahal sebagian
    bahan masih tersisa untuk produksi berikutnya.
-   Kebutuhan bahan dihitung berdasarkan perkiraan resep.
-   Hasil resep tidak selalu persis sama.
-   Penggunaan bahan aktual dapat berbeda dari estimasi.
-   Stok bahan dan kemasan belum tercatat secara terstruktur.
-   Hasil produksi yang tersisa belum tercatat sebagai stok produk jadi.
-   Biaya produksi dan laba belum dihitung secara detail.
-   Biaya seperti LPG dan transport sulit dialokasikan secara tepat ke
    satu produksi.

DapurCalc harus membuat proses tersebut lebih terukur tanpa menciptakan
presisi palsu.

------------------------------------------------------------------------

## 3. Product Goals

### G-001

Memudahkan pengguna mencatat pembelian dan mengetahui stok
bahan/kemasan.

### G-002

Menghitung estimasi kebutuhan bahan berdasarkan resep dan target
produksi.

### G-003

Membandingkan estimasi dengan penggunaan dan hasil produksi aktual.

### G-004

Menghitung biaya berdasarkan material yang digunakan, bukan seluruh
nilai pembelian.

### G-005

Menyimpan hasil produksi sebagai finished goods inventory.

### G-006

Memenuhi order menggunakan finished goods dan mencatat penggunaan
packaging.

### G-007

Memberikan informasi sederhana mengenai stok, biaya produksi,
pendapatan, dan laba.

### G-008

Memberikan pengalaman mobile-first yang mudah digunakan oleh pengguna
non-teknis.

------------------------------------------------------------------------

## 4. Non-Goals / Out of Scope V1

V1 **tidak** mencakup:

-   SaaS / multi-tenant.
-   Multi-business atau multi-branch.
-   Customer account.
-   WhatsApp API/integration.
-   Payment gateway.
-   Delivery management.
-   Payroll.
-   Tax accounting.
-   Full double-entry accounting.
-   AI forecasting.
-   Advanced warehouse management.
-   Automatic packaging rules.
-   Complex supplier management.
-   Native Android/iOS app.
-   Produk finished goods berbasis berat seperti Tekwan/kg.
-   Subscription/billing.
-   Advanced roles and permissions.

Hal-hal tersebut dapat dipertimbangkan setelah V1 diuji.

------------------------------------------------------------------------

## 5. Target User

### Admin / Pemilik Usaha

Satu role pada V1.

Admin dapat:

-   mencatat pembelian;
-   mengelola material;
-   melihat stok;
-   mengelola produk dan resep;
-   mencatat order;
-   merencanakan produksi;
-   mencatat produksi aktual;
-   melakukan fulfillment;
-   mencatat operational expense;
-   melihat laporan sederhana.

Customer tidak memiliki akun. Order dari WhatsApp dimasukkan manual oleh
Admin.

------------------------------------------------------------------------

## 6. Core Mental Model

Workflow utama yang harus terasa sederhana bagi pengguna:

**Catat Belanja → Catat Pesanan → Cek Stok → Hitung Kebutuhan → Produksi
→ Catat Hasil → Penuhi Pesanan → Lihat Stok & Laba**

Secara domain:

**Pembelian → Material Inventory → Production → Finished Goods Inventory
→ Order Fulfillment**

Order menciptakan demand. Production menciptakan finished goods.
Production tidak wajib terikat langsung dengan order tertentu.

------------------------------------------------------------------------

## 7. Units

### FR-001 --- Unit Master

Sistem mendukung minimal:

  Unit              Type         Base Conversion
  ----------------- ---------- -----------------
  gram (g)          WEIGHT                     1
  kilogram (kg)     WEIGHT                  1000
  milliliter (ml)   VOLUME                     1
  liter (L)         VOLUME                  1000
  pcs               QUANTITY                   1

Internal base units:

-   WEIGHT → gram
-   VOLUME → ml
-   QUANTITY → pcs

### BR-001

Unit yang digunakan untuk sebuah material harus kompatibel dengan
`unit_type` base unit material.

### BR-002

Konversi seperti "1 pack = 100 pcs" tidak menjadi global unit conversion
karena isi kemasan berbeda antar material.

Untuk V1, pembelian dapat dimasukkan dalam base-compatible quantity.
Contoh: 1 pack mika berisi 100 pcs dicatat sebagai 100 pcs dengan notes
"1 pack".

------------------------------------------------------------------------

## 8. Materials & Material Inventory

### FR-002 --- Material Master

Material memiliki kategori:

-   `RAW_MATERIAL`
-   `PACKAGING`

Minimal menyimpan:

-   nama;
-   kategori;
-   base unit;
-   current stock;
-   weighted average cost;
-   active/inactive.

Contoh:

-   Ikan Sarden --- RAW_MATERIAL --- gram
-   Tapioka --- RAW_MATERIAL --- gram
-   Minyak --- RAW_MATERIAL --- ml
-   Mika --- PACKAGING --- pcs

### BR-003

Material yang telah digunakan dalam histori tidak di-hard-delete.
Gunakan inactive.

### BR-004

Normal operation tidak boleh menghasilkan negative material stock.

------------------------------------------------------------------------

## 9. Purchases

### FR-003 --- Purchase

Admin dapat membuat pembelian dengan lifecycle:

-   `DRAFT`
-   `POSTED`
-   `VOID`

Purchase dapat memiliki banyak purchase items.

Purchase item minimal menyimpan:

-   material;
-   input quantity;
-   input unit;
-   base quantity;
-   subtotal;
-   notes.

### BR-005

`DRAFT` dapat diedit dan belum mempengaruhi inventory.

### BR-006

Ketika purchase menjadi `POSTED`:

1.  validasi seluruh item;
2.  hitung quantity dalam base unit;
3.  hitung purchase cost per base unit;
4.  hitung weighted average cost baru;
5.  buat material stock movement `IN`;
6.  update `current_stock`;
7.  update `average_cost`;
8.  ubah purchase menjadi `POSTED`.

Seluruh proses harus atomic.

### BR-007 --- Material Weighted Average

Formula:

`new_average_cost = ((old_stock × old_average_cost) + new_purchase_value) / (old_stock + new_quantity)`

### BR-008

Purchase yang sudah POSTED tidak diedit atau di-hard-delete. Koreksi
harus melalui controlled VOID/reversal.

------------------------------------------------------------------------

## 10. Material Stock Ledger

### FR-004 --- Material Stock Movement

Semua perubahan material inventory harus memiliki audit trail.

Movement minimal:

-   material;
-   direction: `IN` / `OUT`;
-   movement reason;
-   quantity dalam base unit;
-   unit cost snapshot;
-   reference type;
-   reference ID;
-   movement date;
-   notes.

Minimal movement reasons:

-   `INITIAL_STOCK`
-   `PURCHASE`
-   `PRODUCTION`
-   `ORDER_FULFILLMENT`
-   `ADJUSTMENT`
-   `WASTE`
-   `REVERSAL`

### BR-009

Movement quantity selalu positif. Direction menentukan efek terhadap
stok.

### BR-010

Stock movement tidak di-hard-delete.

------------------------------------------------------------------------

## 11. Products

### FR-005 --- Product Master

Product minimal memiliki:

-   nama;
-   selling unit;
-   current selling price;
-   current finished-goods stock;
-   weighted average finished-goods cost;
-   active/inactive.

Contoh:

**Pempek Lenjer Sarden --- pcs --- Rp5.000/pcs**

### BR-011

`current_selling_price` adalah harga master saat ini, bukan histori
transaksi.

### BR-012

V1 hanya perlu mendukung finished goods berbasis quantity/pcs. Produk
berbasis kg adalah V2.

------------------------------------------------------------------------

## 12. Recipes

### FR-006 --- Versioned Recipe

Product dan Recipe adalah konsep berbeda.

Satu product dapat memiliki banyak recipe version.

Recipe status:

-   `DRAFT`
-   `ACTIVE`
-   `ARCHIVED`

Recipe minimal memiliki:

-   product;
-   version;
-   estimated yield;
-   yield unit;
-   status;
-   notes.

Recipe item minimal memiliki:

-   material;
-   input unit;
-   quantity;
-   base quantity.

### BR-013

Hanya boleh ada satu ACTIVE recipe per product.

### BR-014

ACTIVE recipe tidak diedit langsung. Perubahan resep dibuat sebagai
versi baru.

### BR-015

Recipe version yang sudah digunakan production harus tetap tersedia
secara historis.

### BR-016

Packaging yang tergantung cara pemenuhan order, seperti mika, bukan
bagian dari food recipe.

### BR-017

Recipe yield adalah estimasi. UI harus menggunakan istilah seperti
`Estimasi`, `±`, atau istilah setara dan tidak menjanjikan hasil pasti.

------------------------------------------------------------------------

## 13. Orders

### FR-007 --- Order

Admin dapat mencatat order secara manual.

Order minimal:

-   customer name;
-   order date;
-   required date (optional);
-   status;
-   notes.

Status:

-   `NEW`
-   `CONFIRMED`
-   `PARTIALLY_FULFILLED`
-   `COMPLETED`
-   `CANCELLED`

### FR-008 --- Order Items

Order dapat memiliki banyak items.

Order item minimal:

-   product;
-   ordered quantity;
-   transaction unit price;
-   subtotal;
-   notes.

### BR-018 --- Selling Price Snapshot

Harga pada order item harus disnapshot. Perubahan harga master product
tidak boleh mengubah order lama.

### BR-019

Order tidak wajib membuat production. Order dapat dipenuhi dari finished
goods yang sudah tersedia.

------------------------------------------------------------------------

## 14. Production Planning

### FR-009 --- Production

Production minimal menyimpan:

-   product;
-   exact recipe version;
-   target quantity;
-   actual yield (setelah produksi);
-   production date;
-   status;
-   notes.

Status:

-   `PLANNED`
-   `IN_PROGRESS`
-   `COMPLETED`
-   `CANCELLED`

Production boleh dibuat tanpa order untuk menambah stok.

### FR-010 --- Estimated Material Requirements

Saat planning, sistem menghitung:

`production_factor = target_quantity / recipe_estimated_yield`

Lalu:

`estimated_material_quantity = recipe_item_base_quantity × production_factor`

Contoh:

Recipe ±32 pcs: - ikan 1000g - gandum 250g - tapioka 1500g

Target 100:

factor = `100 / 32 = 3.125`

Estimasi: - ikan = 3125g - gandum = 781.25g - tapioka = 4687.5g

### BR-020

Estimated quantity dan estimated unit cost harus disimpan sebagai
snapshot.

### BR-021

Planning tidak mengurangi stock.

### FR-011 --- Stock Availability

Sistem menampilkan apakah stock mencukupi dan jumlah shortage.

Availability harus dicek ulang saat production benar-benar
diposting/completed.

------------------------------------------------------------------------

## 15. Actual Production

### FR-012 --- Actual Material Usage

Admin dapat memasukkan actual material usage.

Setiap actual usage dapat memiliki source:

-   `MEASURED`
-   `ESTIMATED`

Jika user tidak mengukur sebuah bahan, aplikasi dapat menawarkan
penggunaan estimated quantity sebagai actual, tetapi harus diberi source
`ESTIMATED`.

### BR-022

UI harus membedakan estimated usage dengan measured actual usage.

### FR-013 --- Actual Yield

Admin mencatat hasil produksi aktual.

Contoh:

-   target = 100 pcs
-   actual yield = 106 pcs

Selisih +6 bukan otomatis laba dan tetap menjadi finished goods
inventory.

### FR-014 --- Complete Production

Saat production COMPLETED:

1.  validasi status;
2.  validasi actual usage;
3.  lock/validasi material stock terbaru;
4.  reject jika stock tidak cukup;
5.  snapshot weighted average cost setiap material;
6.  buat material stock movement OUT;
7.  kurangi material current stock;
8.  hitung actual production cost;
9.  simpan actual yield;
10. hitung production cost per finished unit;
11. masukkan actual yield ke finished goods;
12. update weighted average finished-goods cost;
13. buat product stock movement IN;
14. ubah production menjadi COMPLETED.

Semua harus dilakukan dalam satu database transaction.

### BR-023

Jika salah satu langkah gagal, seluruh transaction harus rollback.

------------------------------------------------------------------------

## 16. Finished Goods Inventory

### FR-015 --- Product Stock Movement

Finished goods memiliki ledger terpisah dari material.

Movement minimal:

-   product;
-   direction;
-   movement reason;
-   quantity;
-   unit cost snapshot;
-   reference type;
-   reference ID;
-   movement date;
-   notes.

Minimal reasons:

-   `INITIAL_STOCK`
-   `PRODUCTION`
-   `ORDER_FULFILLMENT`
-   `ADJUSTMENT`
-   `WASTE`
-   `REVERSAL`

### BR-024

Finished goods normal operation tidak boleh negatif.

### BR-025 --- Finished Goods Weighted Average

Jika finished stock lama dan production baru memiliki cost berbeda,
update average cost menggunakan weighted average.

### BR-026

Production cost per finished unit:

`actual attributable production material cost / actual yield`

Packaging fulfillment tidak dimasukkan ke production cost jika packaging
digunakan saat order fulfillment.

------------------------------------------------------------------------

## 17. Order Fulfillment

### FR-016 --- Fulfillment

Order item dapat dipenuhi satu atau beberapa kali.

Fulfillment minimal:

-   order item;
-   quantity;
-   fulfilled date/time;
-   notes.

Contoh:

Order 100 pcs: - fulfillment 1 = 60 - fulfillment 2 = 40

### BR-027

Total fulfillment tidak boleh melebihi ordered quantity kecuali
requirement tersebut secara eksplisit diubah di masa depan.

### BR-028

Fulfillment harus memvalidasi finished goods stock terbaru sebelum
posting.

### BR-029

Fulfillment yang diposting:

-   membuat product stock movement OUT;
-   mengurangi finished goods stock;
-   menyimpan finished goods unit cost snapshot;
-   memperbarui order status.

Jika fulfilled \> 0 tetapi \< ordered → `PARTIALLY_FULFILLED`.

Jika seluruh items fulfilled → `COMPLETED`.

------------------------------------------------------------------------

## 18. Packaging During Fulfillment

### FR-017 --- Fulfillment Materials

Admin dapat mencatat packaging yang digunakan ketika fulfillment.

Contoh:

100 pempek, 4 per mika → user mencatat 25 mika.

V1 tidak perlu automatic packaging configuration.

Fulfillment material minimal:

-   fulfillment;
-   material;
-   quantity;
-   unit cost snapshot.

### BR-030

Saat fulfillment diposting, packaging stock dikurangi melalui material
stock movement `OUT / ORDER_FULFILLMENT`.

### BR-031

Finished goods OUT dan packaging OUT untuk fulfillment harus diproses
secara atomic.

------------------------------------------------------------------------

## 19. Operational Expenses

### FR-018 --- Operational Expense

Admin dapat mencatat biaya yang tidak dapat dialokasikan secara wajar ke
production/order tertentu.

Contoh:

-   LPG/gas;
-   transport;
-   listrik;
-   biaya operasional lain.

Minimal:

-   date;
-   category/description;
-   amount;
-   notes.

### BR-032

Operational expense tidak dipaksakan masuk recipe atau production unit
cost jika tidak ada metode atribusi yang masuk akal.

------------------------------------------------------------------------

## 20. Costing & Profit

### FR-019 --- Production Cost

Sistem menampilkan minimal:

-   estimated material cost;
-   actual material cost;
-   variance;
-   actual yield;
-   actual cost per finished unit.

### FR-020 --- Order / Fulfillment Profit

Untuk order yang fulfilled, sistem dapat menghitung:

-   revenue berdasarkan transaction price;
-   cost of finished goods yang keluar;
-   packaging cost;
-   attributable cost;
-   production/order profit estimate.

### BR-033

Jangan menyebut angka tersebut sebagai "laba bersih usaha" jika
operational expenses dan komponen akuntansi lain belum seluruhnya
diperhitungkan.

Gunakan istilah seperti: - laba produksi; - laba order; - laba setelah
biaya operasional tercatat, jika konteks laporan periode mendukung.

------------------------------------------------------------------------

## 21. Primary Acceptance Scenario

### AC-001 --- Bu Ani, 100 Pempek

Given: - Product: Pempek Lenjer Sarden - Selling price: Rp5.000/pcs -
Active recipe estimated yield: ±32 pcs - Recipe: ikan 1000g, gandum
250g, tapioka 1500g - Finished goods initial stock: 0

When: 1. Bu Ani orders 100 pcs. 2. Order revenue snapshot = Rp500.000.
3. Production target 100 is created. 4. System estimates required
materials using factor 100/32. 5. Actual usage is entered: - ikan
3200g - gandum 800g - tapioka 4700g 6. Actual yield = 106 pcs. 7.
Production is completed. 8. Material stock decreases by actual usage. 9.
106 pcs enter finished goods. 10. Bu Ani receives 100 pcs. 11. 25 mika
are recorded for fulfillment.

Then: - order becomes COMPLETED; - finished goods stock becomes 6 pcs; -
mika stock decreases 25 pcs; - historical purchase/product/recipe prices
remain unchanged; - production actual cost is reconstructable from
snapshots; - the remaining 6 pcs can fulfill another order.

------------------------------------------------------------------------

## 22. Required Edge Cases

### AC-002

Production menghasilkan kurang dari target. Order dapat menjadi
partially fulfilled dan sisanya dipenuhi kemudian.

### AC-003

Customer cancel sebelum production. Tidak ada stock yang dikurangi hanya
karena production masih PLANNED.

### AC-004

Customer cancel setelah production selesai. Finished goods tidak
dikembalikan menjadi raw material.

### AC-005

Material stock berubah setelah planning. Posting production harus
menggunakan stock terbaru dan gagal jika tidak cukup.

### AC-006

Material purchase price berubah setelah planning. Estimated cost tetap
snapshot lama; actual cost menggunakan cost saat actual consumption
diposting.

### AC-007

Recipe baru diaktifkan. Production lama tetap menunjuk exact recipe
version yang digunakan.

### AC-008

Finished product rusak. Catat product stock movement `OUT / WASTE`.

### AC-009

Production dapat dilakukan tanpa order dan hasilnya menjadi stock.

### AC-010

Order dapat dipenuhi seluruhnya dari existing finished goods tanpa
membuat production baru.

------------------------------------------------------------------------

## 23. MVP Screens

Mobile-first screens minimum:

1.  **Dashboard**
    -   ringkasan order;
    -   low/insufficient stock alert sederhana;
    -   quick actions;
    -   ringkasan produksi/stok.
2.  **Material & Stok**
    -   daftar material;
    -   current stock;
    -   average cost;
    -   movement history.
3.  **Pembelian**
    -   daftar purchase;
    -   create/edit draft;
    -   post purchase;
    -   detail purchase.
4.  **Produk**
    -   product list;
    -   current price;
    -   finished stock;
    -   average cost.
5.  **Resep**
    -   recipe versions;
    -   recipe items;
    -   activate recipe version.
6.  **Pesanan**
    -   order list;
    -   create order;
    -   order detail;
    -   fulfillment history.
7.  **Produksi**
    -   production list;
    -   create planning;
    -   estimated requirements;
    -   stock availability;
    -   input actual usage;
    -   input actual yield;
    -   complete production.
8.  **Fulfillment**
    -   quantity fulfilled;
    -   packaging used;
    -   confirmation before posting.
9.  **Biaya Operasional**
    -   list;
    -   create expense.
10. **Laporan Sederhana**
    -   stock summary;
    -   production cost summary;
    -   revenue/order profit;
    -   operational expenses.

------------------------------------------------------------------------

## 24. UX Requirements

### UX-001

Mobile-first; primary flows nyaman digunakan di smartphone.

### UX-002

Gunakan Bahasa Indonesia yang mudah dipahami.

### UX-003

Jangan tampilkan istilah database seperti foreign key, stock movement
ID, atau weighted-average formula kepada pengguna biasa kecuali
dibutuhkan di detail teknis.

### UX-004

Format uang menggunakan Rupiah.

### UX-005

Tampilkan unit secara manusiawi, misalnya 3.125g dapat ditampilkan
sebagai 3,125 kg jika sesuai.

### UX-006

Bedakan `Estimasi` dan `Aktual` secara jelas.

### UX-007

Berikan warning yang jelas ketika stok tidak cukup.

### UX-008

Transaksi yang mengubah stock harus meminta konfirmasi sebelum posting.

### UX-009

Posted state harus terlihat berbeda dari draft.

### UX-010

Prioritaskan sedikit langkah dan input sederhana daripada dashboard
kompleks.

------------------------------------------------------------------------

## 25. Non-Functional Requirements

### NFR-001 --- Data Integrity

Operasi yang mempengaruhi beberapa ledger/stock harus menggunakan
database transaction.

### NFR-002 --- Concurrency

Posting stock harus aman terhadap race condition. Revalidate dan lock
data inventory yang diperlukan dalam transaction.

### NFR-003 --- Numeric Precision

Gunakan PostgreSQL `NUMERIC/DECIMAL` untuk quantity, unit cost, dan
money. Jangan gunakan floating point untuk financial calculations.

### NFR-004 --- Auditability

Posted inventory-changing transactions harus dapat ditelusuri ke
sumbernya.

### NFR-005 --- Responsive

Aplikasi harus usable pada smartphone modern tanpa horizontal scrolling
pada primary flows.

### NFR-006 --- Performance

Untuk skala satu usaha kecil, primary page/action sebaiknya terasa
responsif pada koneksi mobile normal.

### NFR-007 --- Security

Secrets/database credentials tidak boleh dikirim ke client atau disimpan
di repository.

### NFR-008 --- Validation

Semua mutation harus divalidasi server-side walaupun client-side
validation juga tersedia.

------------------------------------------------------------------------

## 26. Proposed Technical Direction

Gunakan sebagai default kecuali ditemukan blocker nyata:

-   **Framework:** Next.js full-stack
-   **Language:** TypeScript
-   **UI:** Tailwind CSS + shadcn/ui
-   **Validation:** Zod
-   **ORM:** Prisma
-   **Database:** PostgreSQL
-   **Hosted DB:** Supabase PostgreSQL free tier untuk
    development/portfolio testing
-   **Architecture:** Modular monolith
-   **Version control:** Git + GitHub
-   **Primary client:** smartphone browser / installable PWA where
    practical

Deployment provider harus dipilih sesuai free-tier terms yang berlaku
ketika deployment dilakukan. Jangan mengikat domain logic ke provider
hosting tertentu.

------------------------------------------------------------------------

## 27. Data Model Direction

Expected core entities:

### Stage 1 --- Inventory & Purchase

-   `units`
-   `materials`
-   `purchases`
-   `purchase_items`
-   `stock_movements`

### Stage 2 --- Product & Recipe

-   `products`
-   `recipes`
-   `recipe_items`

### Stage 3 --- Order, Production & Fulfillment

-   `orders`
-   `order_items`
-   `productions`
-   `production_materials`
-   `product_stock_movements`
-   `order_fulfillments`
-   `fulfillment_materials`

Additional minimal entity: - `operational_expenses`

Exact schema must follow the business rules in this PRD. Do not add
entities merely for architectural elegance.

------------------------------------------------------------------------

## 28. Feature Priority

### MUST

-   Material/unit management
-   Purchase DRAFT/POSTED flow
-   Material stock ledger
-   Weighted average material cost
-   Products
-   Versioned recipes
-   Orders + price snapshot
-   Production estimation
-   Actual material usage/yield
-   Production posting
-   Finished goods inventory
-   Fulfillment
-   Packaging consumption
-   Operational expenses
-   Basic cost/profit view
-   Mobile-first UI
-   Transaction-safe stock updates

### SHOULD

-   Simple dashboard
-   Stock shortage warnings
-   Waste/adjustment flows
-   Basic period report
-   PWA installability

### COULD

-   CSV export
-   Additional dashboard metrics
-   Improved search/filtering

### WON'T --- V1

Everything explicitly listed in Out of Scope.

------------------------------------------------------------------------

## 29. V2 Candidates

Only consider after V1 user testing:

-   Product inventory and sales by weight, e.g. Tekwan per kg.
-   Better package/purchase-unit conversion.
-   Payment/DP tracking.
-   Production history analytics and average actual yield.
-   Automatic packaging rules.
-   Customer master/history.
-   Improved purchasing/supplier features.
-   Better reports.
-   SaaS/multi-tenant architecture if validated with other businesses.

V2 requirements must be based on actual feedback where possible.

------------------------------------------------------------------------

## 30. Open Questions to Validate During V1

These should not block initial implementation unless they affect a
current feature:

1.  Which ingredients are realistically measured during actual
    production?
2.  How often are orders partially fulfilled in practice?
3.  How often is production performed specifically for stock?
4.  Which operational expenses are useful to record routinely?
5.  Which reports does the user actually look at?
6.  Is PWA installation useful to the user or is browser access
    sufficient?
7.  Which workflow steps feel too slow when used from a phone?

Do not implement speculative solutions before validation.

------------------------------------------------------------------------

## 31. Definition of Done --- DapurCalc V1

V1 is considered complete when:

1.  A fresh database can be migrated and seeded successfully.
2.  Admin can record material purchases and posting updates
    stock/weighted average correctly.
3.  Admin can create Pempek Lenjer Sarden and an active recipe.
4.  Admin can record an order with a selling-price snapshot.
5.  Admin can create production planning and receive estimated material
    requirements.
6.  System warns about insufficient stock.
7.  Admin can enter actual material usage and actual yield.
8.  Completing production atomically reduces raw materials and increases
    finished goods.
9.  Admin can fulfill an order from finished goods.
10. Fulfillment atomically reduces finished goods and packaging.
11. Partial fulfillment works.
12. Remaining finished goods stay available for future orders.
13. Waste/adjustment does not destroy audit history.
14. Basic production cost, revenue, attributable cost, and profit can be
    viewed.
15. Operational expenses can be recorded separately.
16. Historical values remain stable when current prices/recipes later
    change.
17. Primary workflows work comfortably on smartphone.
18. The Bu Ani acceptance scenario passes end-to-end.
19. At least one real user can use the core flow without developer
    intervention.
20. No V2/SaaS feature is required to declare V1 complete.

------------------------------------------------------------------------

## 32. Implementation Milestones for Codex

### Milestone 1 --- Foundation

-   Initialize Next.js/TypeScript.
-   Configure styling/UI foundation.
-   Configure PostgreSQL/Prisma.
-   Environment validation.
-   Base project structure.

### Milestone 2 --- Inventory Core

-   Units.
-   Materials.
-   Purchases.
-   Posting transaction.
-   Material stock ledger.
-   Weighted average tests.

### Milestone 3 --- Product & Recipe

-   Products.
-   Recipe versions/items.
-   Active recipe constraints.
-   Recipe scaling calculation.

### Milestone 4 --- Orders

-   Orders/items.
-   Transaction-price snapshot.
-   Status basics.

### Milestone 5 --- Production

-   Planning.
-   Estimated requirements.
-   Availability checks.
-   Actual usage/yield.
-   Atomic production posting.
-   Finished-goods inventory.

### Milestone 6 --- Fulfillment

-   Partial fulfillment.
-   Finished-goods OUT.
-   Packaging OUT.
-   Atomic fulfillment posting.

### Milestone 7 --- Cost & Operations

-   Operational expenses.
-   Production cost.
-   Order/fulfillment attributable cost.
-   Basic profit/report views.

### Milestone 8 --- Mobile UX & Hardening

-   Mobile-first polish.
-   Validation/errors/loading states.
-   Edge-case tests.
-   Bu Ani end-to-end acceptance test.
-   PWA only if it does not destabilize core MVP.

------------------------------------------------------------------------

## 33. Instructions for Coding Agents

This PRD is the source of truth for DapurCalc V1.

When implementing:

1.  Read this PRD before making architectural decisions.
2.  Do not implement V2, SaaS, or out-of-scope features.
3.  Work one milestone at a time.
4.  Do not silently change business rules.
5.  If a requirement is ambiguous and materially affects data integrity,
    stop and ask.
6.  Prefer simple code over speculative abstractions.
7.  Protect financial and inventory calculations with automated tests.
8.  All stock-changing operations must be server-side and
    transaction-safe.
9.  Preserve historical snapshots.
10. Do not introduce microservices, queues, Redis, or infrastructure not
    required by V1.
11. Keep the UI optimized for a non-technical smartphone user.
12. Treat estimated values as estimates and never relabel them as
    measured actual values.

**Primary implementation goal:** deliver a small, trustworthy
application that the real user can actually use.
