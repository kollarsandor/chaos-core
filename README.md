A rendszer kvantum-inspirált memória- és feladatütemező kernele. Négy fő alrendszert tartalmaz, amelyeket a ChaosCoreKernel fog össze.

MemoryBlock: egy memóriablokk reprezentációja. Tartalmaz 16 bájtos blokk-ID-t (SHA-256 alapú, tartalom+timestamp-ből), 16 bájtos tartalom-hash-t, nyers adatot, hozzáférési számlálót, utolsó hozzáférési időbélyeget, és egy BlockIdSet-et az entanglement-kapcsolatokhoz. A updateAccessTime() frissíti a statisztikát.

ContentAddressableStorage: tartalom-alapú memóriatároló. A store() SHA-256-tal hash-eli a tartalmat, és ha már létezik azonos tartalmú blokk, visszaadja azt (deduplication). Ha nincs elég hely, evictLeastUsed() fut: először a nem-entangled blokkokat lakoltatja ki, majd ha szükséges, az entangled-eket is, hozzáférési szám és utolsó hozzáférési idő alapján rendezve. A findNearestCore() az entanglement-kapcsolatok alapján választja ki a legközelebbi processzormagot.

DynamicTaskScheduler: prioritásos feladatsor (std.PriorityQueue). A scheduleTask() pontozási rendszerrel választ magot: +10 pont, ha a feladat adatfüggősége az adott magon van, +1 pont egyéb esetben, +(1-workload)*5 pont az alacsony terhelésű magoknak.

DataFlowAnalyzer: rögzíti, melyik mag mikor fért hozzá melyik blokkhoz (recordAccess), és ebből analyzeFlow()-val affinitás-statisztikát számol (hány hozzáférés volt melyik magról, normalizálva).

ChaosCoreKernel: az összes fenti komponens koordinátora. Az executeCycle() egy teljes ciklust hajt végre: feladatütemezés → mag-tick → optimizeDataPlacement() (adatblokkok migrálása a legjobb affinitású magra, ha az affinitás > 0.6) → 100 ciklusonként balanceLoad() (túlterhelt magokról blokkokat migrál az alulterhelt magokra). Az executeGraphOnKernel() egy teljes NSIR-gráfot tölt be a kernelbe: minden csomópontot memóriablokkként tárol, az éleket entanglement-kapcsolatokként kezeli. 
