<!-- dcterms:identifier = aspnetcz#285 -->
<!-- dcterms:title = Geografická data v .NET 1: Spatial funkce SQL Serveru 2008 -->
<!-- dcterms:abstract = Jako malé hříbě mne fascinovala analytická geometrie. Idea, že rozličné plošné i prostorové tvary je možno matematicky popsat a elegantními rovnicemi řešit geometrické úlohy typu kde se co protíná, mne prostě fascinovala. Tím spíše, že maje obě přední nohy levé, jsem nikdy nebyl schopen úspěšně řešit úlohy typu "zkonstruujte pravítkem a kružítkem dvě tečny kružnice, protínající se v úhlu tom a tom". Ano, už tehdy mi moji spolužáci říkali, že jsem perverzní – a to ještě nevěděli o mnoha jiných věcech, ke kterým jsem později dospěl. S rozmachem různých geo-technologií jako je GPS, geotagging a geotracking přibývá případů, kdy se nám v databázi rodí data geografické nebo geometrické povahy a před programátory jsou kladeny úlohy jako "vypiš všechny body vzdálené méně než N kilometrů". -->
<!-- np9:categoryId = 1 -->
<!-- x4w:category = Programování -->
<!-- np9:authorId = 1 -->
<!-- np9:authorEmail = michal.valasek@altairis.cz -->
<!-- dcterms:creator = Michal Altair Valášek -->
<!-- np9:serialId = 6 -->
<!-- x4w:serial = Geografická data v .NET -->
<!-- dcterms:created = 2010-06-03T19:06:25.327+02:00 -->
<!-- dcterms:dateAccepted = 2010-06-03T19:06:27+02:00 -->
<!-- x4w:pictureWidth = 150 -->
<!-- x4w:pictureHeight = 150 -->
<!-- x4w:pictureUrl = /perex-pictures/20100603-geograficka-data-v-net-1-spatial-funkce-sql-serveru-2008.png -->

<p>Jako malé hříbě mne fascinovala analytická geometrie. Idea, že rozličné plošné i prostorové tvary je možno matematicky popsat a elegantními rovnicemi řešit geometrické úlohy typu kde se co protíná, mne prostě fascinovala. Tím spíše, že maje obě přední nohy levé, jsem nikdy nebyl schopen úspěšně řešit úlohy typu "zkonstruujte pravítkem a kružítkem dvě tečny kružnice, protínající se v úhlu tom a tom". Ano, už tehdy mi moji spolužáci říkali, že jsem perverzní – a to ještě nevěděli o mnoha jiných věcech, ke kterým jsem později dospěl. S rozmachem různých geo-technologií jako je GPS, geotagging a geotracking přibývá případů, kdy se nám v databázi rodí data geografické nebo geometrické povahy a před programátory jsou kladeny úlohy jako "vypiš všechny body vzdálené méně než N kilometrů". </p>
<p>Již nějakou dobu je všeobecně známo, že Země není placatá. Ačkoliv to má své výhody, z hlediska řešení úloh shora popsaného typu je to poněkud nepraktické, protože si nevystačíme s prostou Pythagorovou větou a je nezbytné užívat poněkud ezoteričtějších matematických konstruktů. Již méně se ví, že Země není ani kulatá, lépe řečeno že nemá tvar koule, ale dosti obtížně popsatelného tvaru, kterému se říká geoid. Jestliže se někdy setkáte se zkratkou WGS84 (a to se jistě setkáte, například dále v tomto článku) pak vězte, že se jedná o demokraticky přijatý matematický model pro určování zeměpisných souřadnic, který sice – jak tomu už u demokratických řešení bývá – není zcela shodný s realitou, ale zato ho všichni používají. Pro nás z toho vyplývá smutná skutečnost, že při řešení výše popsaných problémů si nevystačíme ani se vzorečky, které bychom možná na základě středoškolsných znalostí dali dohromady pro kouli.</p>
<p>Nicméně, netřeba propadat trudnomyslnosti. Výše popsaný úvod je zde především proto, aby vás vyděsil a umožnil vám naplno si vychutnat úlevu ze skutečnosti, že díky Microsoftu a jeho SQL Serveru nebudete muset nic podobného řešit. Veškeré popsané matematické prostocviky totiž za vás udělá zmíněný program, a to od verze 2008 (ve všech edicích, včetně Express).</p>
<blockquote>
<p>Disclaimer: Tento článek je jedním z mnoha mých výletů mimo domovský přístav ASP.NET a vrtám se do věcí, o kterých mnoho nevím. Články v tomto seriálu tedy berte spíše jako volnou inspiraci. Za pomoc při jejich tvorbě (zejména pak pozdějích pokračování s JavaScriptem) děkuji Petru Kaletovi.</p>
</blockquote>
<h2>Datové typy 'geometry' a 'geography'</h2>
<p>SQL Server přináší datové typy <code>geometry</code> a <code>geography</code>. Říká se jim také spatial data types (prostorové datové typy). Datový typ <code>geometry</code> je zcela univerzální typ pro práci s jakýmikoliv daty zadanými pomocí prostorových souřadnic, v podstatě v jakémkoliv referenčním modelu. Tyto typy umějí reprezentovat v podstatě libovolnou prostorovou konstrukci – bod, polygon, trasu…</p>
<p>Já se budu nadále zabývat pouze typem <code>geography</code>, který je speciálním případem shora uvedeného pro potřeby práce s pozemskými zeměpisnými souřadnicemi. </p>
<p>K lokalizaci jakéhokoliv bodu na zemském povrchu potřebujeme znát jeho zeměpisnou šířku (anglicky latitude, obvykle se používá zkratka <code>Lat</code>) a zeměpisnou délku (longitude, zkratka <code>Lon</code> nebo <code>Long</code>). Zeměpisná&nbsp;šířka nabývá hodnot od -90 do 90 a nula leží na rovníku.&nbsp;Zeměpisná délka se pohybuje&nbsp;-180 do 180 s nulou na nultém poledníku. V případě pevninské Evropy budou obě čísla kladná, ČR se nachází přibližně na padesátém stupni severní šířky a čtrnáctém stupni východní délky. </p>
<p>Z vlastní zkušenosti mohu říct, že se při importu dat vyplatí ověřit, že jste souřadnice neprohodili, abyste se pak nedivili jako já, že vám to vychází nějak divně.</p>
<p>Pro další pokusy a ukázky jsem si vybral a do SQL serveru naimportoval data volně dostupná na serveru <a href="http://www.geonames.org/">Geonames</a>. Pro ČR jsou dostupné zeměpisné souřadnice obcí, některých ulic, význačných bodů a podobně. Je jich zhruba 19000, což je pro potřeby našich demonstrací až dost. Na konci tohoto článku najdete odkaz na stažení SQL skriptu, který vytvoří požadovanou strukturu databáze a naplní ji ukázkovými daty.</p>
<p>Začneme vytvořením prázdné databáze <code>Geo</code> a v ní dvou tabulek <code>Classes</code> a <code>GeoPoints</code>:</p>
<div style="background: white; color: black; font-family: consolas, &quot;courier new&quot;, monospace; font-size: 10pt;">
<p style="margin: 0px;"><span style="color: #008000;">-- Vytvořit nové tabulky</span></p>
<p style="margin: 0px;"><span style="color: #0000ff;">CREATE TABLE </span>Classes (</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; ClassCode&nbsp;&nbsp; <span style="color: #0000ff;">char</span>(1)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">NOT NULL</span>,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">Name </span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">varchar</span>(200)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">NOT NULL</span>,</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;">)</p>
<p style="margin: 0px;"><span style="color: #0000ff;">CREATE TABLE </span>GeoPoints(</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; GeoPointId&nbsp; <span style="color: #0000ff;">int IDENTITY</span>(1,1)&nbsp; <span style="color: #0000ff;">NOT NULL</span>,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">Name </span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">varchar</span>(200)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">NOT NULL</span>,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; Class&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">char</span>(1)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">NOT NULL</span>,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; Location&nbsp;&nbsp;&nbsp; geography&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">NOT NULL</span>,</p>
<p style="margin: 0px;">)</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;"><span style="color: #008000;">-- Vytvořit indexy</span></p>
<p style="margin: 0px;"><span style="color: #0000ff;">ALTER TABLE </span>Classes <span style="color: #0000ff;">ADD</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">CONSTRAINT </span>PK_Classes <span style="color: #0000ff;">PRIMARY KEY CLUSTERED </span>(ClassCode)</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;"><span style="color: #0000ff;">ALTER TABLE </span>GeoPoints <span style="color: #0000ff;">ADD</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">CONSTRAINT </span>PK_GeoPoints <span style="color: #0000ff;">PRIMARY KEY CLUSTERED </span>(GeoPointId),</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">CONSTRAINT </span>FK_GeoPoints_Classes <span style="color: #0000ff;">FOREIGN KEY </span>(Class) <span style="color: #0000ff;">REFERENCES </span>dbo.Classes (ClassCode) </p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;"><span style="color: #008000;">-- Vložit data do pomocné tabulky Classes</span></p>
<p style="margin: 0px;"><span style="color: #0000ff;">INSERT </span>[Classes] ([ClassCode], [Name]) <span style="color: #0000ff;">VALUES </span>(N<span style="color: #a31515;">'A'</span>, N<span style="color: #a31515;">'Územní celek'</span>)</p>
<p style="margin: 0px;"><span style="color: #0000ff;">INSERT </span>[Classes] ([ClassCode], [Name]) <span style="color: #0000ff;">VALUES </span>(N<span style="color: #a31515;">'H'</span>, N<span style="color: #a31515;">'Vodní tok'</span>)</p>
<p style="margin: 0px;"><span style="color: #0000ff;">INSERT </span>[Classes] ([ClassCode], [Name]) <span style="color: #0000ff;">VALUES </span>(N<span style="color: #a31515;">'L'</span>, N<span style="color: #a31515;">'Park, rezervace'</span>)</p>
<p style="margin: 0px;"><span style="color: #0000ff;">INSERT </span>[Classes] ([ClassCode], [Name]) <span style="color: #0000ff;">VALUES </span>(N<span style="color: #a31515;">'P'</span>, N<span style="color: #a31515;">'Sídlo'</span>)</p>
<p style="margin: 0px;"><span style="color: #0000ff;">INSERT </span>[Classes] ([ClassCode], [Name]) <span style="color: #0000ff;">VALUES </span>(N<span style="color: #a31515;">'R'</span>, N<span style="color: #a31515;">'Doprava'</span>)</p>
<p style="margin: 0px;"><span style="color: #0000ff;">INSERT </span>[Classes] ([ClassCode], [Name]) <span style="color: #0000ff;">VALUES </span>(N<span style="color: #a31515;">'S'</span>, N<span style="color: #a31515;">'Budova, místo'</span>)</p>
<p style="margin: 0px;"><span style="color: #0000ff;">INSERT </span>[Classes] ([ClassCode], [Name]) <span style="color: #0000ff;">VALUES </span>(N<span style="color: #a31515;">'T'</span>, N<span style="color: #a31515;">'Přírodní úkaz'</span>)</p>
<p style="margin: 0px;"><span style="color: #0000ff;">INSERT </span>[Classes] ([ClassCode], [Name]) <span style="color: #0000ff;">VALUES </span>(N<span style="color: #a31515;">'V'</span>, N<span style="color: #a31515;">'Les'</span>)</p>
</div>
<p>Středem všehomíra je pro nás tabulka <code>GeoPoints</code>. Obsahuje numerický identifikátor, textový název, typ (třídu) objektu a především sloupec <code>Location</code>, který je výše zmíněného typu <code>geography</code>. </p>
<p>&nbsp;</p>
<p>Přímé vkládání dat do tabulky jest činiti například pomocí funkce <code>geography::STPointFromText</code>. Ona na první pohled podezřelá syntaxe je způsobena tím, že spatial funkce zhusta využívají SQLCLR – jsou interně implmentovány v .NET Frameworku a referencovány ze SQL. Syntaxe INSERT dotazů pak bude vypadat následovně:</p>
<div style="background: white; color: black; font-family: consolas, &quot;courier new&quot;, monospace; font-size: 10pt;">
<p style="margin: 0px;"><span style="color: #0000ff;">INSERT INTO </span>GeoPoints (<span style="color: #0000ff;">Name</span>, Class, Location) <span style="color: #0000ff;">VALUES </span>(<span style="color: #a31515;">'Praha'</span>, <span style="color: #a31515;">'P'</span>, geography::STPointFromText(<span style="color: #a31515;">'POINT(14.424132 50.087837)'</span>, 4326))</p>
</div>
<p>První parametr funkce STPointFromText je textové vyjádření souřadnic. Podobně fungují i další funkce, které umožňují vytvářet složitější objekt, jako čáry a polygony. Druhý parametr je referenční model, ke kterému se údaje vztahují. Číslo <code>4326</code> reprezentuje již výše zmíněný model WGS84, se kterým se budete patrně setkávat nejčastěji, protože se typicky používá pro určování GPS souřadnic. </p>
<p>Pozor, z mně neznámých důvodů stringový formát jako první uvádí zeměpisnou délku a poté šířku, zatímco prakticky všude jinde je to naopak.</p>
<p>Data jsou interně ukládána binárně, jako binárku je možné je také vyexportovat a naimportovat zpět, což je výrazně rychlejší, než parsování textu, pokud data už někde máme. Ekvivalentní příkaz k výše uvedenému by byl:</p>
<div style="background: white; color: black; font-family: consolas, &quot;courier new&quot;, monospace; font-size: 10pt;">
<p style="margin: 0px;"><span style="color: #0000ff;">INSERT INTO </span>GeoPoints (<span style="color: #0000ff;">Name</span>, Class, Location) <span style="color: #0000ff;">VALUES </span>(<span style="color: #a31515;">'Praha'</span>, <span style="color: #a31515;">'P'</span>, 0xE6100000010C7B30293E3E0B4940C85F5AD427D92C40)</p>
</div>
<p>Do databáze je možné se potom normálně dotazovat. Pokud chcete souřadnice dostat v nějaké příčetné podobě, musíte použít vlastnosti <code>Latitude</code>, <code>Longitude</code> a nebo metodu <code>ToString</code>:</p>
<div style="background: white; color: black; font-family: consolas, &quot;courier new&quot;, monospace; font-size: 10pt;">
<p style="margin: 0px;"><span style="color: #0000ff;">SELECT</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">Name</span>,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; Location.Lat <span style="color: #0000ff;">AS </span>Latitude, </p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; Location.Long <span style="color: #0000ff;">AS </span>Longitude,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; Location.ToString() <span style="color: #0000ff;">AS </span>LocationString</p>
<p style="margin: 0px;"><span style="color: #0000ff;">FROM </span>GeoPoints</p>
</div>
<p>Datové typy <code>geometry</code> a <code>geography</code> lze též indexovat pomocí takzvaných spatial indexů. Pokud hodláte data nejenom uchovávat, ale též podle nich vyhledávat, je dobrý nápad tento index vytvořit, protože to výrazně zrychlí dobu odezvy. Spatial indexy mají množství optimalizačních parametrů, do kterých nebudu blíže rýpat (primárně proto, že o nich mnoho nevím) takže se spokojím s jednoduchým vytvořením indexu:</p>
<div style="background: white; color: black; font-family: consolas, &quot;courier new&quot;, monospace; font-size: 10pt;">
<p style="margin: 0px;"><span style="color: #0000ff;">SET </span>ANSI_PADDING <span style="color: #0000ff;">ON</span></p>
<p style="margin: 0px;"><span style="color: #0000ff;">CREATE </span>SPATIAL <span style="color: #0000ff;">INDEX </span>IX_Location <span style="color: #0000ff;">ON </span>GeoPoints (Location)</p>
</div>
<h2>Geografické podmínky</h2>
<p>Jakmile máme jednou data v databázi, je snadné páchat elegantní dotazy založené na vzájemné poloze jednotlivých bodů. Nejtypičtějším zadáním je patrně nalézt body v blízkosti zadaných souřadnic. K tomuto účelu lze využít metodu <code>STDistance</code>, která vrátí vzdálenost dvou bodů, v případě typu <code>geography</code> v metrech.</p>
<p>Následujícím dotazem vybereme položky do vzdálenosti <code>@delta</code> metrů od bodu <code>@point</code> – v tomto případě do deseti kilometrů od středu Brna:</p>
<div style="background: white; color: black; font-family: consolas, &quot;courier new&quot;, monospace; font-size: 10pt;">
<p style="margin: 0px;"><span style="color: #0000ff;">DECLARE </span>@delta <span style="color: #0000ff;">AS int</span></p>
<p style="margin: 0px;"><span style="color: #0000ff;">DECLARE </span>@point <span style="color: #0000ff;">AS </span>geography</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;"><span style="color: #0000ff;">SET </span>@delta = 10000</p>
<p style="margin: 0px;"><span style="color: #0000ff;">SET </span>@point = geography::STPointFromText(<span style="color: #a31515;">'POINT(16.607552 49.199541)'</span>, 4326)</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;"><span style="color: #0000ff;">SELECT</span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">Name</span>, </p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; @point.STDistance(Location) <span style="color: #0000ff;">AS </span>Distance,</p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; Location.Lat <span style="color: #0000ff;">AS </span>Latitude, </p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; Location.Long <span style="color: #0000ff;">AS </span>Longitude</p>
<p style="margin: 0px;"><span style="color: #0000ff;">FROM </span>GeoPoints </p>
<p style="margin: 0px;"><span style="color: #0000ff;">WHERE </span>@point.STDistance(Location) &lt;= @delta</p>
<p style="margin: 0px;"><span style="color: #0000ff;">ORDER BY </span>2</p>
</div>
<p>Tento dotaz budete typicky využívat pro zadání typu "najdi nejbližší…". Druhý základní typ dotazu je "najdi body ve vyznačeném území" – například v aktuálně zobrazeném výřezu mapy. V takovém případě podobným způsobem jako bod nadefinujeme polygon (v níže uvedeném příkladu obdélník <code>@rect</code>, odpovídající přibližně ČR) a poté využijeme metodu <code>STIntersects</code>, která vrátí nulu nebo jedničku v závislosti na tom, zda se dva objekty protínají (v našem případě tedy zda je bod uvnitř polygonu):</p>
<div style="background: white; color: black; font-family: consolas, &quot;courier new&quot;, monospace; font-size: 10pt;">
<p style="margin: 0px;"><span style="color: #0000ff;">DECLARE </span>@rect <span style="color: #0000ff;">AS </span>geography</p>
<p style="margin: 0px;"><span style="color: #0000ff;">SET </span>@rect = geography::STPolyFromText(<span style="color: #a31515;">'POLYGON ((11.381836 51.869708, 11.381836 47.613570, 19.621582 47.613570, 19.621582 51.869708, 11.381836 51.869708))'</span>, 4326)</p>
<p style="margin: 0px;">&nbsp;</p>
<p style="margin: 0px;"><span style="color: #0000ff;">SELECT </span></p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; <span style="color: #0000ff;">Name</span>, </p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; Location.Lat <span style="color: #0000ff;">AS </span>Latitude, </p>
<p style="margin: 0px;">&nbsp;&nbsp;&nbsp; Location.Long <span style="color: #0000ff;">AS </span>Longitude</p>
<p style="margin: 0px;"><span style="color: #0000ff;">FROM </span>GeoPoints </p>
<p style="margin: 0px;"><span style="color: #0000ff;">WHERE </span>Location.STIntersects(@rect) = 1</p>
</div>
<p>&nbsp;</p>
<p>Spatial funkce SQL serveru toho samozřejmě umí mnohem víc, další informace najdete například <a href="http://www.microsoft.com/sqlserver/2008/en/us/spatial-data.aspx">na webu Microsoftu</a>. V příštím pokračování se podíváme, jak se na data tohoto typu ptát z .NETu a jak si poradit s výsledkem.</p>
<h2>Příklady ke stažení</h2>
<p>Příklady k tomuto seriálu si můžete stáhnout na <a href="https://www.cdn.altairis.cz/Blog/2010/20100603-GeoSamples.zip">http://www.aspnet.cz/files/20100603-GeoSamples.zip</a> (760 kB).</p>