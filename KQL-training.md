``` Kusto
//------------------------------------------------------------------------------
// Part1 - základní operátory
//------------------------------------------------------------------------------

//------------------------------------------------------------------------------
// volání tabulky
//------------------------------------------------------------------------------
    Event | count //běžné volání

    ["Event"] | count //volání s quoting

    ['Event'] | count //volání s quoting

    workspace("office365securitystack").Event | count //volání napříč workspace

//------------------------------------------------------------------------------
// volání přes cluster
//------------------------------------------------------------------------------
    //volání přes cluster je možné pouze v Azure Data Explorer
    cluster("https://help.kusto.windows.net")
    cluster("https://ade.loganalytics.io/subscriptions/2b47eb3a-fb55-4379-8907-be70b5ef8f3c/resourcegroups/rg-sentinel/providers/microsoft.operationalinsights/workspaces/office365securitystack").database("office365securitystack").BehaviorAnalytics

//------------------------------------------------------------------------------
// volání tabulky skrze funkci, jako proměnnou
//------------------------------------------------------------------------------
    let foo = (tableName:string)
    {
        table(tableName) | count
    };
    foo('Usage')

//------------------------------------------------------------------------------
// vyheledání dat napříč workspace
//------------------------------------------------------------------------------
    workspace("Office365SecurityStack").Usage | count //Volání přes WorkspaceName
    workspace("54af4a30-0c8d-4e6e-a0ec-cd7ee9a1e415").Usage | count //Volání přes ID(GID) prostředku)

//------------------------------------------------------------------------------
// práce se stringem
//------------------------------------------------------------------------------
    print ("Nějaký text")

    print ('Nějaký text')

    print ("Nějaký \\ text")

    print ('Nějaký \\ text')

    print (@"Nějaký \ text")

    print (@'Nějaký \ text')
    
    print x="Moje heslo je"h' superadmin' //je při spuštění vidět, ale pro ostatní je skrytý a nahrazen *

//------------------------------------------------------------------------------
// hledání v datech pomocí fulltext search
//------------------------------------------------------------------------------

Usage | search "SigninLogs" // Bude hledate všechny sloupce v Perf tabulce, kde najde shodu na Memory

Usage | search "signinlogs" // Search není case sensitive by default

Usage | search kind=case_sensitive "SigninLogs" // můžeme však postavit dotaz, tak aby vracel data s case sensitive

search "SigninLogs" //běžný search může trvat nekonečně dlouho (až do timeout)

search in (Usage, Event, Alert) "SigninLogs" // lepší je hledat v konkrétních tabulkách

Usage | search DataType == "SigninLogs" // hledat můžete v konkrétním sloupci a to konkrétní hodnotu

Usage | search DataType:"*Signin*" //hledat je možné také jakoukoliv hodnotu v konkrétním slopupci

Usage | search "SigninLogs" // Hledat můžete přesnou shodu 

Usage | search "**Signin*" // hledat můžete i s využítím wildcard a to napříč

Usage | search * startswith "Signin" // místo zástupného znaku můžete hledat s operátorem startwith

Usage | search * endswith "Bytes" // místo zástupného znaku můžete hledat s operátorem endwith

Usage | search "Sig*ogs" // widlcard lze použít kdekoliv uvnitř hledané hodnoty

Usage | search "Sig*ogs" and ("true" or "MBytes") // vyhledání může být kombinováno s logickými operátory

Usage | search DataType matches regex "[A-Z]" // hledání také podporuje regulérné výrazy RE2

Usage | search "Sig*ogs" | search "*MB*" //můžete i řetězit search, ale pozor na výkon dotazu  (hledat lze ve stringu)

//------------------------------------------------------------------------------
// podmína where
//------------------------------------------------------------------------------
SecurityEvent | where TimeGenerated >= ago(1h) //řeknete, že je potřeba omezení v čase (tedy před 1h)

SecurityEvent | where TimeGenerated  >= ago(1h) and Channel == "Security" // V řetězci je možné použít logickou podmínku

SecurityEvent | where TimeGenerated  >= ago(1h) and Channel == "Security" and toint(Level) > 8 // Podmínek může být mnohem více v jednom dotazu

SecurityEvent | where TimeGenerated >= ago(1h) and (Channel == "Security" or  Channel contains "AppLocker") // Místo logického oprátoru AND, můžete používat i další operátory (napříkld OR)

SecurityEvent | where TimeGenerated  >= ago(1h) and (Channel == "Security" or  Channel contains "AppLocker") and Level == "16" // oprátory je možné kombinovat

SecurityEvent | where TimeGenerated  >= ago(1h) | where (Channel == "Security" or  Channel contains "AppLocker") | where Level == "16" // Podmínku where lze vrstvit, tedy použít více v řetězu

SecurityEvent | where * has "AppLocker" // pomocí where můžete taktéž zastoupit search a vyhledávat fultextově

SecurityEvent | where * hasprefix "Microsoft"  // Můžete hledat na základě konkrétního prefixu (hledá ve všch sloupcích)

SecurityEvent | where * hasprefix "Auditing" // Můžete hledat na základě konkrétního suffixu (hledá ve všch sloupcích)

SecurityEvent | where * contains "Audit" //můžeme hledat na základě konkrétního slova (ve všech sloupcích

SecurityEvent | where Activity matches regex "[0-9] - " // Where také podporuje hledání pomocí RE2 Regex

//------------------------------------------------------------------------------
// has_all operátor
//------------------------------------------------------------------------------

// vyhledá více klíčových slov v konkrétním sloupci
let keywords = datatable (words:string)["Security", "Microsoft"];
SecurityEvent
| where EventSourceName has_all(keywords)

//------------------------------------------------------------------------------
// has_any operátor
//------------------------------------------------------------------------------

// vyhledá jedno z klíčových slov v konkrétním sloupci
let keywords = datatable (words:string)["Security", "Microsoft123"];
SecurityEvent
| where EventSourceName has_any(keywords)

//------------------------------------------------------------------------------
// použtí operátoru Take a Limit
//------------------------------------------------------------------------------

SecurityEvent | take 10 // Take vezme náhodné výsledky z konkrétního logu jako vzorek

SecurityEvent | where TimeGenerated  >= ago(1h) and (Channel == "Security" or  Channel contains "AppLocker") | take 5 // Take může být řetězen s dalšími funkcemi

SecurityEvent | limit 10 // Limit je obdobou hodnoty take

//------------------------------------------------------------------------------
// Count, jak vracet počty
//------------------------------------------------------------------------------

SecurityEvent | count // vrátí počet řádků v tabulce

SecurityEvent | where TimeGenerated  >= ago(1h) and (Channel == "Security" or  Channel contains "AppLocker") | count // count je možné používat společně s dalšími podmínkami a řetězit

//------------------------------------------------------------------------------
// Použití tabulkového operátoru Summarize
//------------------------------------------------------------------------------

SecurityEvent | summarize count() by Channel //provede sečtení řádků, podle konkrétního sloupce

SecurityEvent | summarize count() by Channel, Level // můžete sčítat na základě více unikátníh kombinací sloupců

SecurityEvent | summarize CustomCount = count() by Channel, Level // během sčítání můžete pojmenovat součtový sloupec názvy sloupců

SecurityEvent | summarize NumberOfEntries=count() , UniqueChannels=dcount(Channel) by Channel // V summarize je možné dělat více výpočetních operací

SecurityEvent | summarize NumberOfEntries=count() by bin(TimeGenerated, 1h) // summarize umožňuje rozložení do časové řady

SecurityEvent | summarize NumberOfEntries=count() by Channel, bin(TimeGenerated, 1h) // Bin je také možné použít na třídění podle počtu hodnot nějakého sloupce nebo sloupců

Usage | summarize NumberOfRowsAtThisPercentLevel=count() by bin(Quantity,0.005) //Bin lze použít i pro jiné hodnoty než jen pro časovou hodnotu (například je možné pouít pro hodnotu)

//------------------------------------------------------------------------------
// Extend
//------------------------------------------------------------------------------

SecurityEvent | where Channel == "Security" | extend Hodnota = 1000 // Extend rozšiřuje pole o nové hodnoty

SecurityEvent | where Channel == "Security" | extend Hodnota = 1000 / 5 // Extend rozšiřuje pole o nové hodnoty. Může obsahovat výpočet

SecurityEvent | extend Hodnota1 = 1000 , polovinaHondnoty = 1000/2 // Extend může být použit pouze jednou a rozšířit více hodnot

SecurityEvent | extend ObjectCounter = strcat("Byl nalezen security event pro - ", AccountType, " - ", Account) // extend může být použitý i pro textové hodnoty

SecurityEvent | where Channel == "Security" | extend Hodnota = toscalar((SecurityEvent | where Channel == "Security" | count)) + toscalar((Usage | count)) // Může obsahovat hodnotu získanou ze scalární funkce

//------------------------------------------------------------------------------
// Project
//------------------------------------------------------------------------------

SecurityEvent | project Account, AccountType, AccountName, Level, TimeGenerated // project funguje jako select v T-SQL, tedy omezí výsledek na konkrétní sloupce

SecurityEvent | project Account, AccountType, AccountName, Level, TimeGenerated | extend MujText = strcat("byla překročena hodnota ",1000) // Kombinace Extend s projekt

SecurityEvent | project Account, AccountType, AccountName, Level, TimeGenerated, MujText = strcat("byla překročena hodnota ",1000) //project může zasoupit extend

SecurityEvent | project-away Account, AccountType, AccountName, Level, TimeGenerated //project-away provede vyloučení konkrétních sloupců z výsledku

SecurityEvent | project-rename Ucet = Account | limit 10 //sloupce můžeme taktéž přejmenovat, ale ponechat kompletní výsledek

//------------------------------------------------------------------------------
// Distinct
//------------------------------------------------------------------------------

SecurityEvent | distinct Account // Distinct vrací unikátní hodnoty na základě zvoleného sloupce nebo sloupců

SecurityEvent | distinct Account, AccountType //je možné zvolit pro výsledek více unikátních kombinací sloupců

//------------------------------------------------------------------------------
// Top
//------------------------------------------------------------------------------

// Top vrací jen omezený počet řádků ze shora tabulky
SecurityEvent | top 20 by TimeGenerated desc

//------------------------------------------------------------------------------
// print
//------------------------------------------------------------------------------

// print může sloužit k zobrazení výsledku. Print může zobrazit pouze scalarní hodnotu (tedy jedno číslo nebo buňku)
print "Hello World"

// je možné používat kalkulace
print 11 * 3

// nebo pracovat s proměnnou ve výsledku a tedy pojmenovat výsledný sloupec
print TheAnswerToLifeTheUniverseAndEverything = 21 * 2

//------------------------------------------------------------------------------
// práce s časem pomocí now a ago
//------------------------------------------------------------------------------

print now() //vrátí aktuální čas

print ago(1d)           // před 1 dnem

print ago(1h)           // před 1 hodinou

print ago(1m)           // před 1 minutou

print ago(1s)           // před 1 sekundou 

print ago(1ms)          // před 1 milisekundou

print ago(1microsecond) // před 1 mikrosekundou

print ago(1tick)        // před 1 nanosekundou

print ago(365d) // hodnota může být jakákoliv, tedy není nutné kalkulovat přes 1 minutu

print ago(12h)

print ago(10m)

print ago(90s)

// je možné volat i negativní hodnoty v ago a dostat se tak do budoucnosti
print ago(-1d)       // zítra

print ago(-365d)     // 1 rok do bdoucnosti

print ago(-1h)       // 1 hodinu do budoucnosti od now()

//------------------------------------------------------------------------------
// práce s datem
//------------------------------------------------------------------------------
    print now()

    print datetime(2015-12-31 23:59:59.9)

    print datetime("2015-12-31 23:59:59.9")

    print ago(14d)

    print now(-14d)

    print now() - ago(14d)

    print now() - datetime(2015-12-31 23:59:59.9)

//------------------------------------------------------------------------------
// práce s časem
//------------------------------------------------------------------------------

    print result1 = 1d / 1s

    print result2 = time(1d) / time(1s)

    print result3 = 24 * 60 * time(00:01:00) / time(1s)

    print result = totimespan("00:01:00")

    print result = time("00:01:00")

//------------------------------------------------------------------------------
// sort (aka order)
//------------------------------------------------------------------------------

// sort utřídí výsledek, stejně jako order
Perf | where TimeGenerated > ago(15m) | where CounterName == "Avg. Disk sec/Read" and InstanceName == "C:"| project Computer, TimeGenerated , ObjectName, CounterName, InstanceName, CounterValue | sort by Computer

// sort může řadit i různé sloupce za sebou
Perf | where TimeGenerated > ago(15m) | where CounterName == "Avg. Disk sec/Read" and InstanceName == "C:"| project Computer, TimeGenerated , ObjectName, CounterName, InstanceName, CounterValue | sort by Computer, TimeGenerated 

// můžeme každý soupec řadit buď vzestupně nebo sestupně
Perf | where TimeGenerated > ago(15m) | where CounterName == "Avg. Disk sec/Read" and InstanceName == "C:"| project Computer, TimeGenerated , ObjectName, CounterName, InstanceName, CounterValue | sort by Computer asc, TimeGenerated asc

// nebo řazení můžeme mixovat
Perf | where TimeGenerated > ago(15m) | where CounterName == "Avg. Disk sec/Read" and InstanceName == "C:"| project Computer, TimeGenerated , ObjectName, CounterName, InstanceName, CounterValue | sort by Computer desc, TimeGenerated asc

//------------------------------------------------------------------------------
// Extract
//------------------------------------------------------------------------------

Perf | where ObjectName == "LogicalDisk" and InstanceName matches regex "[A-Z]:" | project Computer , CounterName , extract("[A-Z]:", 0, InstanceName) //Provedeme extrakci písmene jednotky ze sloupce Instance Name

Perf | where ObjectName == "LogicalDisk" and InstanceName matches regex "[A-Z]:" | project Computer , CounterName , extract("([A-Z]:)", 1, Computer) // můžeme také vracet jen část výsledku, kdy hodnota v druhém parametru určí kolik znaků od zadu bude vynecháno 

 //extract může občas nahradit substring 
print Text = "abcdefghijklmnopqrstuvwxyz" | project extract("^.{2,2}(.{4,4})", 1, Text)
print Text = "abcdefghijklmnopqrstuvwxyz" | project substring(Text, 2, 4)

//------------------------------------------------------------------------------
// Parse
//------------------------------------------------------------------------------

//parse slouží k parsování string hodnoty uvnitř nějakého sloupce (* nakonci zahodí celý konec, * na začátku zahodí začátek až po první vyhledání
SecurityEvent | where EventID == 4702 | parse EventData with * '<Data Name="SubjectUserSid">' ParseSubjectUserSid '</Data>' * | project Computer, ParseSubjectUserSid

//------------------------------------------------------------------------------
// Datetime / timespan arithmetic
//------------------------------------------------------------------------------

// jak dlouho od aktuálního času byl zpracován konkrétní event
Perf | where CounterName == "Avg. Disk sec/Read" | where CounterValue > 0 | take 100 | extend HowLongAgo=( now() - TimeGenerated ) | project Computer, CounterName, CounterValue, TimeGenerated, HowLongAgo 

// můžeme také udělat odpočet od konkrétního data
Perf
| where CounterName == "Avg. Disk sec/Read"
| where CounterValue > 0
| take 100                       
| extend HowLongAgo=( now() - TimeGenerated )
       , TimeSinceStartOfYear=( TimeGenerated - datetime(2021-01-01) )
| project Computer 
        , CounterName
        , CounterValue  
        , TimeGenerated 
        , HowLongAgo 
        , TimeSinceStartOfYear 

//v čase je možné používat kalkulace, v tomto případě kolik hodin od začátku roku
Perf
| where CounterName == "Avg. Disk sec/Read"
| where CounterValue > 0
| take 100                       // done just to give us a small dataset to demo
| extend HowLongAgo=( now() - TimeGenerated )
       , TimeSinceStartOfYear=( TimeGenerated - datetime(2021-01-01) )
| extend TimeSinceStartOfYearInHours=( TimeSinceStartOfYear / 1h )
| project Computer 
        , CounterName
        , CounterValue  
        , TimeGenerated 
        , HowLongAgo 
        , TimeSinceStartOfYear 
        , TimeSinceStartOfYearInHours

// můžeme měřit i délku trvání mezi dvěma časy
Usage
| extend Duration=( EndTime - StartTime )
| project Computer
        , StartTime 
        , EndTime 
        , Duration 

//------------------------------------------------------------------------------
// StartOf...
//------------------------------------------------------------------------------

//můžeme se podívat kdy byl začátek dne
Event
| where TimeGenerated >= ago(2d)
| extend DayGenerated = startofday(TimeGenerated)
| project Source
        , TimeGenerated 
        , DayGenerated 

// můžeme také počítat kolik událostí bylo v daném dnu (stejné jako bin(TimeGenerated,1d)
Event
| where TimeGenerated >= ago(7d)
| extend DayGenerated = startofday(TimeGenerated)
| project Source
        , DayGenerated 
| summarize EventCount=count() 
         by DayGenerated

// můžeme získat i informaci o začátku měsíce
Event
| where TimeGenerated >= ago(365d)
| extend MonthGenerated = startofmonth(TimeGenerated)
| project Source
        , MonthGenerated 
| summarize EventCount=count() 
         by MonthGenerated
         , Source

// stejně tak bychom mohli počítat i počet událostí v každém měsíci
Event
| where TimeGenerated >= ago(365d)
| extend MonthGenerated = startofmonth(TimeGenerated)
| project Source
        , MonthGenerated 
| summarize EventCount=count() 
         by MonthGenerated
         , Source
| sort by MonthGenerated desc
        , Source asc

// stejná funkce existuje pro rok i týden
Event
| where TimeGenerated >= ago(365d)
| extend YearGenerated = startofyear(TimeGenerated)
| project Source
        , YearGenerated 
| summarize EventCount=count() 
         by YearGenerated
         , Source
| sort by YearGenerated desc
        , Source asc

Event
| where TimeGenerated >= ago(365d)
| extend WeekGenerated = startofweek(TimeGenerated)
| project Source
        , WeekGenerated 
| summarize EventCount=count() 
         by WeekGenerated
         , Source
| sort by WeekGenerated desc
        , Source asc 

//------------------------------------------------------------------------------
// EndOf...
//------------------------------------------------------------------------------

// můžeme se dívat i na to kdy končí konkrétní den
Event
| where TimeGenerated >= ago(7d)
| extend DayGenerated = endofday(TimeGenerated)
| project Source
        , DayGenerated 
| summarize EventCount=count() 
         by DayGenerated
         , Source
| sort by DayGenerated desc
        , Source asc 

// můžeme se dívat i na to kdy končí konkrétní týden
Event
| where TimeGenerated >= ago(365d)
| extend WeekGenerated = endofweek(TimeGenerated)
| project Source
        , WeekGenerated 
| summarize EventCount=count() 
         by WeekGenerated
         , Source
| sort by WeekGenerated desc
        , Source asc 

// můžeme se dívat i na to kdy končí konkrétní měsíc
Event
| where TimeGenerated >= ago(365d)
| extend MonthGenerated = endofmonth(TimeGenerated)
| project Source
        , MonthGenerated 
| summarize EventCount=count() 
         by MonthGenerated
         , Source
| sort by MonthGenerated desc
        , Source asc 

// můžeme se dívat i na to kdy končí konkrétní rok
Event
| where TimeGenerated >= ago(365d)
| extend YearGenerated = endofyear(TimeGenerated)
| project Source
        , YearGenerated 
| summarize EventCount=count() 
         by YearGenerated
         , Source
| sort by YearGenerated desc
        , Source asc 

//------------------------------------------------------------------------------
// Between
//------------------------------------------------------------------------------

// podmínka where může být řízena operátorem between, tedy načíst data jen v určitém intervalu hodnot
Perf
| where CounterName == "% Free Space" 
| where CounterValue between ( 70.0 .. 100.0 )

// stejně tak je možné načítat hodnoty mezi intervaly data
Perf
| where CounterName == "% Free Space" 
| where TimeGenerated between ( datetime(2018-04-01) .. datetime(2018-04-03) )

// obdobně bychom mohli určité hodnoty vyloučit
Perf
| where CounterName == "% Free Space" 
| where CounterValue !between ( 0.0 .. 69.9999 )

//------------------------------------------------------------------------------
// todynamic
//------------------------------------------------------------------------------

// vezeme JSON stringovou hodnotu, která je umístěna v jednom sloupci a budeme ji měnit na dynamický objekt a rozdělovat

// zde je příklad rozšířené vlatnosti, která obhauje JSON objekt ve formátu stringu
// column ExtendedProperties
// "{
//   ""Alert Start Time (UTC)"": ""2018/04/02 10:57:59.7540414"",
//   ""Source"": ""IP Address: 175.195.219.31"",
//   ""Non-Existent Users"": ""84"",
//   ""Existing Users"": ""1"",
//   ""Failed Attempts"": ""85"",
//   ""Successful Logins"": ""0"",
//   ""Successful User Logons"": ""[]"",
//   ""Account Logon Ids"": ""[]"",
//   ""Failed User Logons"": ""DEMOUSER"",
//   ""End Time UTC"": ""4/2/2018 11:57:54 AM"",
//   ""ActionTaken"": ""Detected"",
//   ""resourceType"": ""Virtual Machine"",
//   ""ServiceId"": ""fb2ebac8-5667-4a74-b9cb-5dba27c9faeb"",
//   ""ReportingSystem"": ""Azure"",
//   ""OccuringDatacenter"": ""southcentralus""
// }"

// poté co zkonvertujeme hodnotu do dynamické hodnoty, pak z ní můžeme vyčítat jednotlivé hodnoty
SecurityAlert
| where TimeGenerated >ago(31d)
| extend ExtProps=todynamic(ExtendedProperties)
| project AlertName 
        , TimeGenerated 
        , ExtProps["Alert Start Time (UTC)"]
        , ExtProps["Source"]
        , ExtProps["Non-Existent Users"]
        , ExtProps["Existing Users"]
        , ExtProps["Failed Attempts"]
        , ExtProps["Successful Logins"]
        , ExtProps["Successful User Logons"]
        , ExtProps["Account Logon Ids"]
        , ExtProps["Failed User Logons"]
        , ExtProps["End Time UTC"]
        , ExtProps["ActionTaken"]
        , ExtProps["resourceType"]
        , ExtProps["ServiceId"]
        , ExtProps["ReportingSystem"]
        , ExtProps["OccuringDatacenter"]

// jednotlivé sloupce si také můžeme přejmenovat
SecurityAlert
| where TimeGenerated >ago(31d)
| extend ExtProps=todynamic(ExtendedProperties)
| project AlertName 
        , TimeGenerated 
        , AlertStartTime = ExtProps["Alert Start Time (UTC)"]
        , Source = ExtProps["Source"]
        , NonExistentUsers = ExtProps["Non-Existent Users"]
        , ExistingUsers = ExtProps["Existing Users"]
        , FailedAttempts = ExtProps["Failed Attempts"]
        , SuccessfulLogins = ExtProps["Successful Logins"]
        , SuccessfulUserLogins = ExtProps["Successful User Logons"]
        , AccountLogonIds = ExtProps["Account Logon Ids"]
        , FailedUserLogins = ExtProps["Failed User Logons"]
        , EndTimeUTC = ExtProps["End Time UTC"]
        , ActionTaken = ExtProps["ActionTaken"]
        , ResourceType = ExtProps["resourceType"]
        , ServiceId = ExtProps["ServiceId"]
        , ReportingSystem = ExtProps["ReportingSystem"]
        , OccuringDataCenter = ExtProps["OccuringDatacenter"]

// pokud obsahuje název mezery, pak musíme použít quotation, jinak není nutné s objektem pracovat v rámci quotation
SecurityAlert
| where TimeGenerated >ago(31d)
| extend ExtProps=todynamic(ExtendedProperties)
| project AlertName 
        , TimeGenerated 
        , AlertStartTime = ExtProps["Alert Start Time (UTC)"]
        , Source = ExtProps.Source
        , NonExistentUsers = ExtProps["Non-Existent Users"]
        , ExistingUsers = ExtProps["Existing Users"]
        , FailedAttempts = ExtProps["Failed Attempts"]
        , SuccessfulLogins = ExtProps["Successful Logins"]
        , SuccessfulUserLogins = ExtProps["Successful User Logons"]
        , AccountLogonIds = ExtProps["Account Logon Ids"]
        , FailedUserLogins = ExtProps["Failed User Logons"]
        , EndTimeUTC = ExtProps["End Time UTC"]
        , ActionTaken = ExtProps.ActionTaken
        , ResourceType = ExtProps.resourceType
        , ["Service Id"] = ExtProps.ServiceId
        , ReportingSystem = ExtProps.ReportingSystem
        , OccuringDataCenter = ExtProps.OccuringDatacenter

//volání je prováděno v objektech, vždy přes ., tady můžeme předpokládat s následujícím vnořováním objektů ExtProps.Level1.Level2

//------------------------------------------------------------------------------
// format_datetime / format_timespan
//------------------------------------------------------------------------------

// formátování času může být prováděno do různých výslekdů (podle potřeby práce s datem)
Perf
| take 100
| project CounterName 
        , CounterValue 
        , TimeGenerated 
        , format_datetime(TimeGenerated, "y-M-d")
        , format_datetime(TimeGenerated, "yyyy-MM-dd")
        , format_datetime(TimeGenerated, "MM/dd/yyyy")
        , format_datetime(TimeGenerated, "MM/dd/yyyy hh:mm:ss tt")
        , format_datetime(TimeGenerated, "MM/dd/yyyy HH:mm:ss")
        , format_datetime(TimeGenerated, "MM/dd/yyyy HH:mm:ss.ffff")
        
// podporované syntaxe
//    d - Day, 1 to 31
//   dd - Day, 01 to 31
//    M - Month, 1 to 12
//   MM - Month, 01 to 12
//    y - Year, 0 to 9999
//   yy - Year, 00 to 9999
// yyyy - Year, 0000 to 9999

// hodiny, minuty a sekundy pro format_timespan mohou být také různé
//  h - Hour, 1 to 12
// hh - Hour, 01 to 12
//  H - Hour, 1 to 23
// HH - Hour, 01 to 23
//  m - Minute, 0 to 59
// mm - Minute, 00 to 59
//  s - Second, 0 to 59
// ss - Second, 00 to 59
// tt - am/pm

// můžeme používat různé separátory pro zápisy času a data
// / - : , . _ [ ] nebo mezeru

// format_timespan formats a timespan. 
// nastavneí základního formátu a konverze do timespan
Perf
| take 100
| project CounterName 
        , CounterValue 
        , TimeGenerated 
        , format_timespan(totimespan(TimeGenerated), "hh:mm:ss")

// Timespans are typically the result of datetime math
Perf
| where TimeGenerated between ( ago(1d) .. ago(1h) )
| take 100                       // done just to give us a small dataset to demo
| extend TimeGen = now() - TimeGenerated
| project CounterName 
        , CounterValue 
        , TimeGenerated 
        , TimeGen 
        , format_timespan(TimeGen, "hh:mm:ss")
        , format_timespan(TimeGen, "HH:mm:ss")
        , format_timespan(TimeGen, "h:m:s")
        , format_timespan(TimeGen, "H:m:s")

// f/F může být použit pro zobrazení části sekundy. f/F není case sensitive
Perf
| take 100
| extend TimeGen = now() - TimeGenerated 
| project CounterName 
        , CounterValue 
        , TimeGenerated 
        , TimeGen 
        , format_timespan(TimeGen, "HH:mm:ss.f")
        , format_timespan(TimeGen, "HH:mm:ss.F")
        , format_timespan(TimeGen, "HH:mm:ss.ff")
        , format_timespan(TimeGen, "HH:mm:ss.FF")
        , format_timespan(TimeGen, "HH:mm:ss.fff")
        , format_timespan(TimeGen, "HH:mm:ss.FFF")
        , format_timespan(TimeGen, "HH:mm:ss.ffff")
        , format_timespan(TimeGen, "HH:mm:ss.FFFF")
        , format_timespan(TimeGen, "HH:mm:ss.fffff")
        , format_timespan(TimeGen, "HH:mm:ss.FFFFF")
        , format_timespan(TimeGen, "HH:mm:ss.ffffff")
        , format_timespan(TimeGen, "HH:mm:ss.FFFFFF")
        , format_timespan(TimeGen, "HH:mm:ss.fffffff")
        , format_timespan(TimeGen, "HH:mm:ss.FFFFFFF")

//------------------------------------------------------------------------------
// datetime_part
//------------------------------------------------------------------------------

// Z výsledku můžeme extrahovat i část data (den, měsíc, rok, atd..)
Perf
| take 100
| project CounterName 
        , CounterValue 
        , TimeGenerated 
        , year = datetime_part("year", TimeGenerated)
        , quarter = datetime_part("quarter", TimeGenerated)
        , month = datetime_part("month", TimeGenerated)
        , weekOfYear = datetime_part("weekOfYear", TimeGenerated)
        , day = datetime_part("day", TimeGenerated)
        , dayOfYear = datetime_part("dayOfYear", TimeGenerated)
        , hour = datetime_part("hour", TimeGenerated)
        , minute = datetime_part("minute", TimeGenerated)
        , second = datetime_part("second", TimeGenerated)
        , millisecond = datetime_part("millisecond", TimeGenerated)
        , microsecond = datetime_part("microsecond", TimeGenerated)
        , nanosecond = datetime_part("nanosecond", TimeGenerated)

// můžeme si tedy zobrazit počet událostí za každou hodinu
Event
| where TimeGenerated >= ago(7d)
| extend HourOfDay = datetime_part("hour", TimeGenerated)
| project HourOfDay 
| summarize EventCount=count() 
         by HourOfDay
| sort by HourOfDay asc 

//------------------------------------------------------------------------------
// Case
//------------------------------------------------------------------------------

// case slouží k vytváření popisku na základě splnění podmínky
Perf
| where CounterName == "% Free Space"
| extend FreeLevel = case( CounterValue < 80, "Critical"
                         , CounterValue < 85, "Danger"
                         , CounterValue < 90, "Look at it"
                         , "You're OK!"
                         )
| project Computer
        , CounterName
        , CounterValue 
        , FreeLevel 

// výsledek pak můžeme sečíst a zobrazi agregovaný
Perf
| where CounterName == "% Free Space"
| extend FreeLevel = case( CounterValue < 80, "Critical (Less than 10% free disk space)"
                         , CounterValue < 85, "Danger (10% to 30% free disk space)"
                         , CounterValue < 90, "Look at it (30% to 50% free disk space)"
                         , "You're OK! (More than 50% free disk space)"
                         )
| summarize ComputerCount=count() 
         by FreeLevel


//------------------------------------------------------------------------------
// iif
//------------------------------------------------------------------------------

// iff je podmínkou ve struktuře if/then/else 
Perf
| where CounterName == "% Free Space"
| extend FreeState = iif( CounterValue < 90
                        , "You might want to look at this"
                        , "You're OK!"
                        )
| project Computer
        , CounterName
        , CounterValue 
        , FreeState 

// podmínka může být použita i s datem
Perf
| where CounterName == "% Free Space"
| where TimeGenerated between ( ago(60d) .. now() )
| extend CurrentMonth = iif( datepart("month", TimeGenerated) == datepart("month", now()) 
                           , "Current Month"
                           , "Past Months"
                           )
| project Computer
        , CounterName
        , CounterValue 
        , CurrentMonth
        , TimeGenerated  

//------------------------------------------------------------------------------
// isempty / isnull
//------------------------------------------------------------------------------

// s null hodnotou se pracuje špatně, nelze ji vytvořit a musí vzniknout, tedy ani podmínku where nelze aplikovat tak, že se bude == hodnotě null, ale je možné použít podmínku isnull nebo isempty

// isempty ve stringové hodnotě
Perf
| where isempty( InstanceName )
| count

Perf
| where TimeGenerated >= ago(1h)
| extend InstName = iif( isempty(InstanceName)
                       , "NO INSTANCE NAME"
                       , InstanceName
                       )
| project Computer
        , TimeGenerated 
        , InstanceName 
        , InstName 
        , ObjectName 
        , CounterName 

// isnull vrátí hodnotu kde je null, třeba v důsledku šaptné deklarace hodnoty (zápis string do sloupce integer)
Perf
| where isnull( SampleCount )
| count

Perf
| where TimeGenerated >= ago(1h)
| extend SampleCountNull = iif( isnull(SampleCount) 
                              , "No Sample Count"
                              , tostring(SampleCount) 
                              )
| project Computer 
        , CounterName 
        , SampleCount 
        , SampleCountNull 

//------------------------------------------------------------------------------
// Split
//------------------------------------------------------------------------------

// split umožní rozdělit hodnotu a vždy vytvoří dynamický objekt (array)
Perf
| take 100
| project Computer 
        , CounterName 
        , CounterValue 
        , CounterPath 
        , CPSplit = split(CounterPath, "\\")

// array je možné oříznout a vrátit jen konkrétní hodnotu řádku z array, výsledkem je sada dynamických hodnot
Perf
| take 100
| extend myComputer = split(CounterPath, "\\", 2) 
       , myObjectInstance = split(CounterPath, "\\", 3)
       , myCounterName = split(CounterPath, "\\", 4)
| project Computer 
        , ObjectName 
        , CounterName 
        , InstanceName 
        , myComputer 
        , myObjectInstance
        , myCounterName
        , CounterPath

// můžeme si také z array volat konkrétní hodnoty (pozor zde se . neuvádí), výsledkem je opět dynamická hodnota
Perf
| take 100
| extend CounterPathArray = split(CounterPath, "\\") 
| extend myComputer = CounterPathArray[2] 
       , myObjectInstance = CounterPathArray[3]
       , myCounterName = CounterPathArray[4]
| project Computer 
        , ObjectName 
        , CounterName 
        , InstanceName 
        , myComputer 
        , myObjectInstance
        , myCounterName
        , CounterPath

//------------------------------------------------------------------------------
// String Operators
//------------------------------------------------------------------------------

// stringová hodnota v podmínce bez case sestitive operátoru
Perf 
| take 100
| where CounterName contains "BYTES"

// stringová hodnota v podmínce s case sestitive operátoru
Perf 
| take 100
| where CounterName contains_cs "BYTES"

//také můžeme používat zápor a tedy omezit aby nevracel konkrétní hodnoty (bez case sensitive)
Perf 
| take 100
| where CounterName !contains "Bytes"

//in nám povolí hledat hodnoty v array
Perf
| take 1000
| where CounterName in ("Disk Transfers/sec", "Disk Reads/sec", "Avg. Disk sec/Write") 

//negace hodnoty in
Perf
| take 100
| where CounterName !in ( "Disk Transfers/sec"
                        , "Disk Reads/sec"
                        , "Avg. Disk sec/Write"
                        ) 

//------------------------------------------------------------------------------
// strcat
//------------------------------------------------------------------------------

//strcat nám povolí spojovat hodnoty do jednoho sloupce
Perf
| take 100
| extend CompObjCounter = strcat(Computer, " - ", ObjectName, " - ", CounterName) 
| project CompObjCounter 
        , TimeGenerated 
        , CounterValue 

//strcat můžeme použít i společně s datem, pak se z výsledku stane string místo datetime
Perf
| where CounterName == "% Free Space"
| where TimeGenerated between ( ago(12m) .. now() )
| extend MonthName = case( datetime_part("month", TimeGenerated) ==  1, "Jan "
                         , datetime_part("month", TimeGenerated) ==  2, "Feb "
                         , datetime_part("month", TimeGenerated) ==  3, "Mar "
                         , datetime_part("month", TimeGenerated) ==  4, "Apr "
                         , datetime_part("month", TimeGenerated) ==  5, "May "
                         , datetime_part("month", TimeGenerated) ==  6, "Jun "
                         , datetime_part("month", TimeGenerated) ==  7, "Jul "
                         , datetime_part("month", TimeGenerated) ==  8, "Aug "
                         , datetime_part("month", TimeGenerated) ==  9, "Sep "
                         , datetime_part("month", TimeGenerated) == 10, "Oct "
                         , datetime_part("month", TimeGenerated) == 11, "Nov "
                         , datetime_part("month", TimeGenerated) == 12, "Dec "
                         , "Unknown Month"
                         )
| extend DateText = strcat( MonthName
                          , datetime_part("day", TimeGenerated)
                          , ", "
                          , datetime_part("year", TimeGenerated) 
                          ) 
| project Computer 
        , CounterName 
        , CounterValue 
        , TimeGenerated 
        , MonthName
        , DateText  

//------------------------------------------------------------------------------
// arg_max / arg_min
//------------------------------------------------------------------------------

//arg_max najde poslední hodnotu v tabulce a vrátí k ní všechny (*) hodnoty přes shluknutí pomocí summarize
Perf
| summarize arg_max(CounterValue, *) by CounterName
| sort by CounterName asc

// můžeme vrátit jen konkrétní slopce do výsledku 
Perf
| summarize arg_max(CounterValue, Computer, ObjectName) by CounterName
| sort by CounterName asc

// arg_min vrátí nejvzdálenější hodnotu a k ní požadované sloupce
Perf
| project CounterName, CounterValue 
| summarize arg_min(CounterValue, *) by CounterName
| sort by CounterName asc

//------------------------------------------------------------------------------
// Makeset / Makelist
//------------------------------------------------------------------------------

// Makeset - Creates a array of json objects by flattening a heirarcy. 
//make_set nebo makeset vytvoří z konkrétních hodnot array. Do tohoto array vrátí jen unikátní hodnoty, jako kdyby proved distinct
Perf
| summarize Counters = makeset(CounterName) by ObjectName

Perf
| summarize Counters = make_set(CounterName) by ObjectName

// můžeme si například vytvořit seznam počítačů, které překročili nějakou hodnotu
Perf
| where CounterName == "% Free Space"
    and CounterValue <= 90
| summarize Computers = makeset(Computer)

// Makelist je podobné jako makeset, ale s tím rozdílem, že se vrátí přesný počet hodnot, který je v tabulce výsledku (neřeší se distinct)
Perf
| where CounterName == "% Free Space"
    and CounterValue <= 90
| summarize Computers = makelist(Computer)

// v makeset a makelist je možné nastavit druhým parametrem limit na počet výsledků
Perf
| where CounterName == "% Free Space"
    and CounterValue <= 90
| summarize ComputersWithLimit = makelist(Computer, 10),ComputersWithoutLimit = makelist(Computer)

//------------------------------------------------------------------------------
// mvexpand
//------------------------------------------------------------------------------

//mvexpand umožní rozbalit hodnoty, které jsou v array a vytvoří je jako samostatné řádky
Perf
| where CounterName == "% Free Space"
    and CounterValue <= 90
| summarize Computers = makeset(Computer)
| mvexpand Computers

SecurityAlert
| where TimeGenerated > ago(90d)
| extend ExtProps=todynamic(ExtendedProperties)
| mvexpand ExtProps
| project TimeGenerated 
        , DisplayName 
        , AlertName 
        , AlertSeverity 
        , ExtProps 

//------------------------------------------------------------------------------
// Percentiles
//------------------------------------------------------------------------------

//Percentil představuje, jak si hodnota z oblasti čísel vede vzhledem k těmto číslům, neboli mám například čísla od 0 do 100 a pokud si vyberu číslo 50, tak 50 % čísel je na tom lépe než číslo 50 a 50 % čísel je na tom hůře. Takže číslo 50 je 50. percentilu. Tj. máte-li čísla seřazeny do řady vzestupně, hodnota 50 leží přesně uprostřed.
Perf
| where CounterName == "Available MBytes"
| summarize percentiles(CounterValue, 5, 50, 95) by Computer

// ve výsledku se tedy říká, že:
// 5%  ze zpracovaných záznamů dosahuje hodnoty 2,237 nebo nižší
// 50% ze zpracovaných záznamů dosahuje hodnoty 2,311.25 nebo nižší
// 95% ze zpracovaných záznamů dosahuje hodnoty 2,350.5 nebo nižší

// výsledek lze taktéž volat přejmenovávat skrze project, project-rename
Perf
| where CounterName == "Available MBytes"
| summarize percentiles(CounterValue, 5, 50, 95) by Computer
| project-rename Percent05 = percentile_CounterValue_5
               , Percent50 = percentile_CounterValue_50 
               , Percent95 = percentile_CounterValue_95 

// přejmenovat sloupce výsledku můžeme i přímo v summarize
Perf
| where CounterName == "Available MBytes"
| summarize (Percent05, Percent50, Percent95) 
          = percentiles(CounterValue, 5, 50, 95) 
         by Computer


// percentile lze použít jako array a tedy vásledek rátit do jednoho sloupce jako array
Perf
| summarize percentiles_array(CounterValue, 5, 50, 95) by CounterName

// tento array pak můžeme rozbalit pomocí mvexpand
Perf
| summarize percentarray = percentiles_array(CounterValue, 5, 50, 95) by CounterName
| mvexpand percentarray

//------------------------------------------------------------------------------
// dcount
//------------------------------------------------------------------------------

// dcount nám poví unikátní počet řádků 

// počet unikátních řádků na základě definovaných sloupců
SecurityEvent 
| distinct EventID, Activity

//count a dcount pro výpočet u konkrétní události je stejný
Event
| where TimeGenerated >= ago(90d)
| distinct Computer, EventID
| summarize EventTypeCount = count(EventID), dcount(EventID) by Computer
| sort by Computer asc

SecurityEvent
| where TimeGenerated >= ago(90d)
| summarize dcount(EventID) by Computer
| sort by Computer asc

// druhý parametr určuje úroveň přesnosti
// 0 = Least accurate, 1.6% error
// 1 = Default, balances accuracy and time, 0.8% error level
// 2 = Accurate but slow, 0.4% error
// 3 = Extra accurate but slowest, 0.28% error level

// 0 = nízká přesnost, 1.6% error
SecurityEvent
| where TimeGenerated >= ago(90d)
| summarize dcount(EventID, 0) by Computer
| sort by Computer asc

// 1 = výchozí přesnost, vyvýžení mezi přesností a časem, 0.8% error level (není nutné udávat, jedná se o výchozí přesnost)
SecurityEvent
| where TimeGenerated >= ago(90d)
| summarize dcount(EventID, 1) by Computer
| sort by Computer asc

// 2 = vysoká přesnost (bude velmi pomalé na počítání), 0.4% error
SecurityEvent
| where TimeGenerated >= ago(90d)
| summarize dcount(EventID, 2) by Computer
| sort by Computer asc

// 3 = velice vysoká přesnost (velmi pomalé na výpočet), 0.28% error level
SecurityEvent
| where TimeGenerated >= ago(90d)
| summarize dcount(EventID, 3) by Computer
| sort by Computer asc

//------------------------------------------------------------------------------
// dcountif
//------------------------------------------------------------------------------

// dcountif je stejné jako dcount, ale povoluje kombinaci s podmínkou
SecurityEvent
| where TimeGenerated >= ago(90d)
| summarize dcountif( EventID
                    , EventID in (4625, 4688, 4624, 4672, 4670, 4689, 4634, 4674)
                    ) 
         by Computer
| sort by Computer asc

// dcountif má stejné nastavení přesnosti jako dcount
// 0 = Least accurate, 1.6% error
// 1 = Default, balances accuracy and time, 0.8% error level
// 2 = Accurate but slow, 0.4% error

// 0 = nízká přesnost, 1.6% error
SecurityEvent
| where TimeGenerated >= ago(90d)
| summarize dcountif( EventID
                    , EventID in (4625, 4688, 4624, 4672, 4670, 4689, 4634, 4674)
                    , 0
                    ) 
         by Computer
| sort by Computer asc

// pro porovnání je zde uveden stejný příkaz, jen pomocí countif
SecurityEvent
| where TimeGenerated >= ago(90d)
| distinct Computer, EventID
| summarize EventTypeCount = countif( EventID in ( 4625, 4688, 4624, 4672
                                                 , 4670, 4689, 4634, 4674
                                                 )
                                    ) by Computer
| sort by Computer asc

//------------------------------------------------------------------------------
// countif
//------------------------------------------------------------------------------

// stejně jako dcountif, countif povoluje přidání podmínky (hodnoty s 0 nejsou zobrazeny)
Perf 
| summarize RowCount = countif(CounterName contains "Bytes") by CounterName
| sort by CounterName asc

// k porovnání bez podmínky (zde je podporována 0 ve výsledku)
Perf 
| where CounterName contains "Bytes" 
| summarize count() by CounterName
| sort by CounterName asc

// řádek s 0 můžeme v pomdínce snadno odstranit
Perf 
| summarize RowCount = countif(CounterName contains "Bytes") by CounterName
| where RowCount > 0
| sort by CounterName asc

//------------------------------------------------------------------------------
// pivot
//------------------------------------------------------------------------------

// pivot nebo kontingenční tabulka nám otočí data a změní určené řádky na sloupce a pak dopočítá potřebné hodnoty
Event
| project Computer, EventLevelName 
| evaluate pivot(EventLevelName)
| sort by Computer asc

//------------------------------------------------------------------------------
// top-nested
//------------------------------------------------------------------------------

// top-nested does nested measurements. 

// top-nested vypočte počet řádků na základě vybraného sloupce, pak provede sort a následně verzme top 3
Perf 
| top-nested 3 of ObjectName by ObjectCount = count() 
, top-nested 3 of CounterName by CounterNameCount = count() 
| sort by ObjectName asc
        , CounterName asc

// jednotlivé výsledky můžeme vrstvit vedle sebe
Perf 
| top-nested 5 of ObjectName by ObjectCount = count() 
, top-nested 5 of CounterName by CounterNameCount = count() 
, top-nested 5 of InstanceName by InstanceCount = count()
| sort by ObjectName asc
        , CounterName asc
        , InstanceName asc

// top-nested podporuje mnoho typů agregací, jako například:
// sum(), count(), max(), min(), dcount(), avg(), percentile(), percentilew(), 
Perf 
| top-nested 5 of ObjectName by ObjectSum = sum(CounterValue)
, top-nested 5 of CounterName by CounterNameSum = sum(CounterValue) 
, top-nested 5 of InstanceName by InstanceSum = sum(CounterValue)
| sort by ObjectName asc
        , CounterName asc
        , InstanceName asc 

// ttop nested povolí přidat do samosatného řádku hodnotu, která bude zobrazovat počet ostatních výsledků
Perf 
| top-nested 3 of ObjectName with others = "All Other Objects" 
          by ObjectCount = count() 
| sort by ObjectName asc

// levelů můžeme přidat více a pak každý sloupec může mít vlastní řádek, kde budou všechny ostatní hodnoty
Perf 
| top-nested 3 of ObjectName 
        with others = "All Other Objects" 
          by ObjectCount = count() 
, top-nested 3 of CounterName 
        with others = "All Other Counters" 
          by CounterNameCount = count() 
| sort by ObjectName asc
        , CounterName asc

//------------------------------------------------------------------------------
// max / min
//------------------------------------------------------------------------------

// max je jednoduché, vrátí hodnotu která je maximální podle určeného požadavku
Perf
| where CounterName == "Free Megabytes"
| summarize max(CounterValue)

//může pracovat i s časem
Perf
| where CounterName == "Free Megabytes"
| summarize max(TimeGenerated)

// min je stejné, jen zobrazí minimální hodnotu z výsledku
Perf
| where CounterName == "Free Megabytes"
| summarize min(CounterValue)

//stejně tak s časem
Perf
| where CounterName == "Free Megabytes"
| summarize min(TimeGenerated)

//------------------------------------------------------------------------------
// sum / sumif
//------------------------------------------------------------------------------

// sum je podovné jako min, max, count, atd... jen sum provede sočítání výsledku hodnot SUMA
Perf
| where CounterName == "Free Megabytes"
| summarize sum(CounterValue)

// sumif provede stejné operace jako SUM jen s tím rozdílem, že může obsahovat podmínku
Perf
| summarize sumif(CounterValue, CounterName == "Free Megabytes")

//------------------------------------------------------------------------------
// any
//------------------------------------------------------------------------------

// any vybere náhodnou hodnotu a tu vrátí
Perf
| summarize any(*)

// můžeme taky vybrat konkrétní sloupec a z něho se vrátí náhodná hodnoty
Perf
| summarize any(Computer) 

// náhodná hodnota může být z konkrétních sloupců
Perf
| summarize any(ObjectName, CounterName, CounterValue)
```