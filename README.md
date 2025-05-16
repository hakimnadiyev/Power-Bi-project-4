Power Bi project 4
RFM analizi
Bu layihə müştəri davranışlarını analiz etmək və vizuallaşdırmaq üçün Power BI istifadə edilərək hazırlanmışdır. 
Bu dashboard vasitəslə kateqoriyalara ayıraraq daha effektiv qerarlar qəbul edə bilərik.Potensial müştərilər üçün əlavə promosyonlar,İtirilmiş müştərilər üçün yeni kampaniyalar təşkil edərək yenidən cəlb edilməsi və s

Əsas Xüsusiyyətlər
Ümumi gəlir və müştəri sayı
Müştəri kateqoriyalarına görə 
Satışların zaman üzrə trend analizi

İstifadə Edilən DAX Formulları

Previous month = 
CALCULATE(
    [Total_Revenues],
    PREVIOUSMONTH('Tarix'[Date])
)       #Trend analizleri üçün

Total_Revenue = 
CALCULATE(
    SUM('Row'[Revenue]),
    ALLEXCEPT('Row', 'Row'[Customer ID])
)      # 1 Customer ID də çox sayda invoice olduğu üçün customer İD yə görə filter tətbiq etmək zərurəti yaranır


Recency_score = 
var Recency_days=
var Son_alish_tarixi=
    CALCULATE(
        MAX('Row'[InvoiceDate]),
        ALLEXCEPT('Row','Row'[Customer ID])
    )
    return 
    DATEDIFF(Son_alish_tarixi,DATE(2023,5,30),DAY)
    RETURN
    SWITCH(
        TRUE(),
        Recency_days<=30,5,
        Recency_days<=60,4,
        Recency_days<=90,3,
        Recency_days<=120,2,
        1)  		# Müştərinin son alış tarixindən keçən günü hesablamaq üçün # Data köhnə olduğu üçün ən son alış nəzərə alınaraq DATE(2023,5,30) qeyd etdim


	Frequency_score = 
VAR tezlik=
    CALCULATE(
        DISTINCTCOUNT('Row'[Invoice]),
        ALLEXCEPT('Row','Row'[Customer ID])
    )
RETURN 
    SWITCH(
        TRUE(),
        tezlik <= 50,1,
        tezlik<=100,2,
        tezlik<=150,3,
        tezlik<=200,4,
        5
    )  # Müştərinin alış veriş tezliklərini nəzərə alaraq rank verildi



 Monetary_score = 
VAR CustomerRevenue =
    CALCULATE(
        SUM('Row'[Revenue]),
        ALLEXCEPT('Row', 'Row'[Customer ID])
    )

VAR CustomerRevenueTable =
    SUMMARIZE(
        'Row',
        'Row'[Customer ID],
        "TotalRevenue", CALCULATE(SUM('Row'[Revenue]))
    )

VAR P1 = PERCENTILEX.INC(CustomerRevenueTable, [TotalRevenue], 0.2)
VAR P2 = PERCENTILEX.INC(CustomerRevenueTable, [TotalRevenue], 0.4)
VAR P3 = PERCENTILEX.INC(CustomerRevenueTable, [TotalRevenue], 0.6)
VAR P4 = PERCENTILEX.INC(CustomerRevenueTable, [TotalRevenue], 0.8)

RETURN
SWITCH(
    TRUE(),
    CustomerRevenue <= P1, 1,
    CustomerRevenue <= P2, 2,
    CustomerRevenue <= P3, 3,
    CustomerRevenue <= P4, 4,
    5
)					# Müştərinin alış etdiyi məbləğə görə rank verdim


Customer_segment = 
VAR R = 'Row'[Recency_score]
VAR F = 'Row'[Frequency_score]
VAR M = 'Row'[Monetary_score]
RETURN
SWITCH(
    TRUE(),
    R >= 4 && F >= 4 && M >= 4, "Loyal müştəri",
    R >= 4 && F >= 3 && M >= 4, "Potensial loyal",
    R >= 4 && F <= 2 && M >= 2, "Yüksələn müştəri",
    R <= 2 && F <= 3 && M <= 3, "Passiv müştəri",
    R = 1 && F = 1 && M = 1, "İtirilmiş müştəri",
    "Digər müştəri"
)				# Yekunda əldə olunan RFM_scorlarina əsasən Müştərilər segmentləşdirildi


Qarşılaşdığım çətinliklər

Müştəridən əldə olunan gəlir böyük olduğuna görə hissələrə manual olaraq bölə bilmədim bu səbəbdən Power Bi var ve percetilex funksiyalarindan istifadı edərək 5 hissəyə böldüm və bu deyerlere esasən rank təyin etdim


Gələcəkdə planladığım yeniliklər

Python vasitəsilə data analizi və vizuallaşdırılması
Satış proqnozlaşdırılması (Forecasting)
ROI (Investment Gəliri) analizləri
İnsan Resursları analizi
Biznəsə fayda verəcək və inkişaf edə biləcəyim daha çox analiz metodlari


