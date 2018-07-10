<!-- dcterms:identifier = aspnetcz#178 -->
<!-- dcterms:title = Trvale udržitelné mailování z webových aplikací -->
<!-- dcterms:abstract = Poslat e-mail z ASP.NET je snadné. Ale jak znění automaticky generovaných zpráv snadno modifikovat? A lokalizovat? A zařídit, aby všechny maily měly nějakou "štábní kulturu"? -->
<!-- np9:categoryId = 1 -->
<!-- x4w:category = Programování -->
<!-- np9:authorId = 1 -->
<!-- np9:authorEmail = michal.valasek@altairis.cz -->
<!-- dcterms:creator = Michal Altair Valášek -->
<!-- dcterms:created = 2007-12-17T10:52:42.59+01:00 -->
<!-- dcterms:dateAccepted = 2007-12-17T10:52:42.59+01:00 -->

<p>Odeslat e-mail z ASP.NET je snadné. Pokud máte aplikaci, která generuje větší množství různých typů mailů, a chcete je nějak rozumně spravovat, lokalizovat do jiných jazyků a podobně, úkol už není zdaleka tak jednoduchý. Ve svých aplikacích používám třídu, která řeší všechny problémy, které jsem až doposud řešil.</p> <h2>Čeho chceme dosáhnout</h2> <p>Jaké problémy řešíme v případě automaticky generovaných e-mailů?</p> <p><strong>Lokalizace.</strong> Pokud je rozhraní aplikace v několika jazycích, měla by zde být možnost automaticky posílat lokalizované zprávy. A to například včetně jména a e-mailové adresy odesílatele.</p> <p><strong>Konzistence.</strong> Zprávy z jednoho webu by měly mít nějakou &quot;štábní kulturu&quot;, aby uživatel snadno poznal, čeho se daný web týká. U automaticky generovaných mailů je také pravděpodobnější, že budou omylem odchyceny antispamovým filtrem. Je tedy dobrý nápad uživateli jejich přijímání usnadnit, aby si mohl například nastavit nějaká vhodná pravidla pro whitelisting. Například všechny zprávy z tohoto webu mají na začátku subjectu text <code>[www.aspnet.cz]</code>.</p> <p>Nezaškodí také, aby maily věly nějaké jednotné zápatí (signaturu), kdežtě napíšete takové informace jako který web to odeslal, jak kontaktovat provozovatele a možná i populární větičku &quot;<em>Tato zpráva byla odeslána automaticky, neodpovídejte na ni.</em>&quot;</p> <p><strong>Trvalá udržitelnost.</strong> Texty předmětných zpráv je třeba čas od času měnit. Někdy musejí být modifikovatelné uživatelsky (v tom případě je třeba je vecpat do databáze), jindy může změnu provést programátor. Ale i v tomto případě je vhodné, aby nemusel editovat C# kód, kde se nepřehledně skládá text mailu.</p> <p><strong>Personalizace.</strong> Do stejné šablony potřebujeme doplnit - a odpovídajícím způsobem formátovat - měnící se data.</p> <h2>Řešení</h2> <p>Moje řešení využívá resource soubory v ASP.NET - RESX. Resources obecně slouží k ukládání různých typů dat. Do resource lze uložit stringy, obrázky, ikony, zvuky... K těmto pak lze programově přistupovat několika způsoby.</p> <p>Resources se používají ve světě ASP.NET převážně pro lokalizaci. Přepínáním vlastnosti <code>CurrentUiCulture</code> aktuálního threadu můžeme volit, jaká jazyková verze se použije.</p> <p>Přístup k resource souborům je rychlý a pohodlný. V konečném důsledku se distribuují ve formě přídavných DLL souborů, ve zkompilované formě.</p> <p>Použití RESX souborů tedy z aktualizací nevyviňuje programátora -- pouze mu poněkud zjednodušuje práci, protože má všechny zprávy na jednom místě.</p> <h3>Vytvoření RESX souboru a práce s ním</h3> <p>Resource soubory v prostředí .NET jsou XML soubory, které jsou zpracovávány při kompilaci. Je možno k nim přistupovat strongly-typed způsobem a nebo pomocí Resource Manageru.</p> <p>Resource soubory musejí být uloženy v adresáři <code>~/App_GlobalResources/</code>. Vytvořte si jej ve své aplikaci a přidejte do něj ve Visual Studiu položku typu <em>Resource File</em> a nazvěte ji <code>MailMessages.resx</code>.</p> <p>Poklepáním otevřete vizuální editor, což je grid se třemi sloupci: <em>Name</em>, <em>Value</em> a <em>Comment</em>. Name je klíč, pod nímž později bude daná hodnota dostupná. Value je vlastní hodnota. Comment je komentář, který se programově nevyužívá, ale může sloužit jako pomůcka pro překladatele při překládání do cizího jazyka. Můžete v něm obecně uvést třeba konkrétní kontext slova, aby překladatel věděl, zda má &quot;volume&quot; překládat jako &quot;hlasitost&quot; nebo jako &quot;svazek&quot; a nedopadlo to jako ve Windows. Vytvořte si novou položku, pojmenujte ji <code>MailSenderName</code> a zadejte do ní nějakou hodnotu, která se použije jako jméno odesílatele e-mailů.</p> <p>Nyní můžete k této hodnotě programově přistupovat tak, že napíšete <code>Resources.MailMessages.MailSenderName</code>. .NET automaticky vytvoří proxy třídu a vygeneruje vlastnosti tak, aby odpovídaly položkám resource souboru.</p> <p>Druhá možnost je, že budete k hodnotám přistupovat pomocí resource manageru a jeho metody <code>GetString</code> takto: <code>string s = Resources.MailMessages.ResourceManager.GetString(&quot;MailSenderName&quot;)</code>.</p> <p>V našem případě budeme využívat obě dvě tyto varianty. Pro přístup k pevně definovaným položkám použijeme strongly-typed metodu, pro přístup k obecným šablonám metodu <code>GetString</code>.</p> <h3>Základní nastavení</h3> <p>Nejprve v resource souboru definujeme několik obecných položek pro všechny zprávy:</p> <ul> <li><code>MailSenderName</code> - zobrazované jméno odesílatele, např. <code>ASPNET.CZ</code>  <li><code>MailSenderEmail</code> - e-mailová adresa odesílateke, např. <code>www-daemon@aspnet.cz</code>.  <li><code>MailEncodingName</code> - název kódování, v němž bude e-mail odeslán, např. <code>iso-8859-2</code>. V zásadě bychom mohli tuto položku pominout a všechny zprávy posílat v UTF-8. To by ale znamenalo zbytečný nárůst jejich objemu, protože by se kódovaly pomocí Base64. Použitím single-byte kódování umožníme použití v daném případě efektivnější metody Quoted Printable.  <li><code>MailSubjectFormatString</code> - formátovací řetězec, který se použije pro vytvoření předmětu zprávy. Na místo, kam má být vložen subject, umístěte <code>{0}</code>. Například: <code>[www.aspnet.cz] {0}</code>.  <li><code>MailBodyFormatString</code> - formátovací řetězec pro text zprávy, vytvořený dle výše uvedeného vzoru. Pro vložení zalomení řádku použijte klávesovou kombinaci <code>Shift+Enter</code>.</li></ul> <p>Poté definujeme statickou třídu <code>Mailer</code> a v ní následující metody:</p> <div style="font-size: 12pt; background: white; color: black; font-family: consolas, courier new"> <p style="margin: 0px"><span style="color: blue">public</span> <span style="color: blue">static</span> <span style="color: blue">void</span> SendMail(<span style="color: blue">string</span> recipient, <span style="color: blue">string</span> subject, <span style="color: blue">string</span> body) {</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: green">// Validate arguments</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (recipient == <span style="color: blue">null</span>) <span style="color: blue">throw</span> <span style="color: blue">new</span> <span style="color: #2b91af">ArgumentNullException</span>(<span style="color: #a31515">&quot;recipient&quot;</span>);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (<span style="color: blue">string</span>.IsNullOrEmpty(recipient)) <span style="color: blue">throw</span> <span style="color: blue">new</span> <span style="color: #2b91af">ArgumentException</span>(<span style="color: #a31515">&quot;Value cannot be null or empty string.&quot;</span>, <span style="color: #a31515">&quot;recipient&quot;</span>);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; SendMail(<span style="color: blue">new</span> <span style="color: blue">string</span>[] { recipient }, subject, body);</p> <p style="margin: 0px">}</p> <p style="margin: 0px">&nbsp;</p> <p style="margin: 0px"><span style="color: blue">public</span> <span style="color: blue">static</span> <span style="color: blue">void</span> SendMail(<span style="color: blue">string</span>[] recipients, <span style="color: blue">string</span> subject, <span style="color: blue">string</span> body) {</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: green">// Validate arguments</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (recipients == <span style="color: blue">null</span>) <span style="color: blue">throw</span> <span style="color: blue">new</span> <span style="color: #2b91af">ArgumentNullException</span>(<span style="color: #a31515">&quot;recipients&quot;</span>);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (subject == <span style="color: blue">null</span>) <span style="color: blue">throw</span> <span style="color: blue">new</span> <span style="color: #2b91af">ArgumentNullException</span>(<span style="color: #a31515">&quot;subject&quot;</span>);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (body == <span style="color: blue">null</span>) <span style="color: blue">throw</span> <span style="color: blue">new</span> <span style="color: #2b91af">ArgumentNullException</span>(<span style="color: #a31515">&quot;body&quot;</span>);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (<span style="color: blue">string</span>.IsNullOrEmpty(subject)) <span style="color: blue">throw</span> <span style="color: blue">new</span> <span style="color: #2b91af">ArgumentException</span>(<span style="color: #a31515">&quot;Value cannot be null or empty string.&quot;</span>, <span style="color: #a31515">&quot;subject&quot;</span>);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (<span style="color: blue">string</span>.IsNullOrEmpty(body)) <span style="color: blue">throw</span> <span style="color: blue">new</span> <span style="color: #2b91af">ArgumentException</span>(<span style="color: #a31515">&quot;Value cannot be null or empty string.&quot;</span>, <span style="color: #a31515">&quot;body&quot;</span>);</p> <p style="margin: 0px">&nbsp; pro</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">using</span> (<span style="color: #2b91af">MailMessage</span> msg = <span style="color: blue">new</span> <span style="color: #2b91af">MailMessage</span>()) {</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: green">// Prepare message</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; msg.From = <span style="color: blue">new</span> <span style="color: #2b91af">MailAddress</span>(Resources.<span style="color: #2b91af">MailMessages</span>.MailSenderEmail, Resources.<span style="color: #2b91af">MailMessages</span>.MailSenderName);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; msg.Subject = <span style="color: blue">string</span>.Format(Resources.<span style="color: #2b91af">MailMessages</span>.MailSubjectFormatString, subject);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; msg.Body = <span style="color: blue">string</span>.Format(Resources.<span style="color: #2b91af">MailMessages</span>.MailBodyFormatString, body).Replace(<span style="color: #a31515">&quot;\\n&quot;</span>, <span style="color: #a31515">&quot;\r\n&quot;</span>);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; msg.SubjectEncoding = System.Text.<span style="color: #2b91af">Encoding</span>.GetEncoding(Resources.<span style="color: #2b91af">MailMessages</span>.MailEncodingName);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; msg.BodyEncoding = System.Text.<span style="color: #2b91af">Encoding</span>.GetEncoding(Resources.<span style="color: #2b91af">MailMessages</span>.MailEncodingName);</p> <p style="margin: 0px">&nbsp;</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: green">// Send messages</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #2b91af">SmtpClient</span> mx = <span style="color: blue">new</span> <span style="color: #2b91af">SmtpClient</span>();</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">foreach</span> (<span style="color: blue">string</span> recipient <span style="color: blue">in</span> recipients) {</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (<span style="color: blue">string</span>.IsNullOrEmpty(recipient)) <span style="color: blue">continue</span>;</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; msg.To.Clear();</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; msg.To.Add(recipient);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mx.Send(msg);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; }</p> <p style="margin: 0px">}</p></div> <p>Nyní máme k dispozici metody pro obecné odeslání e-mailu s dodržením výše definovaých parametrů, a to v overloadech ro zaslání jednomu nebo více příjemcům.</p> <h3>Šablony mailů</h3> <p>Dále je záhodno vyřešit ještě vlastní text mailů. I k tomu využijeme resources. Pro každý typ mailu budeme mít dvě položky, jednu pro subject a jednu pro text, jmenovat se budou <code><em>NázevŠablony</em>Subject</code> a <code><em>NázevŠablony</em>Body</code>.</p> <p>Předmět zprávy je v mé implementaci statický, text zprávy opět může obsahovat zástupky ve formátu <code>{číslo}</code>, které budou potom použity při formátování výsledného řetězce.</p> <p>K odeslání šablonovaného mailu slouží metoda <code>SendTemplatedMail</code>:</p> <div style="font-size: 12pt; background: white; color: black; font-family: consolas, courier new"> <p style="margin: 0px"><span style="color: blue">public</span> <span style="color: blue">static</span> <span style="color: blue">void</span> SendTemplatedMail(<span style="color: blue">string</span> recipient, <span style="color: blue">string</span> templateName, <span style="color: blue">params</span> <span style="color: blue">object</span>[] args) {</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: green">// Validate arguments</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (recipient == <span style="color: blue">null</span>) <span style="color: blue">throw</span> <span style="color: blue">new</span> <span style="color: #2b91af">ArgumentNullException</span>(<span style="color: #a31515">&quot;recipient&quot;</span>);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (<span style="color: blue">string</span>.IsNullOrEmpty(recipient)) <span style="color: blue">throw</span> <span style="color: blue">new</span> <span style="color: #2b91af">ArgumentException</span>(<span style="color: #a31515">&quot;Value cannot be null or empty string.&quot;</span>, <span style="color: #a31515">&quot;recipient&quot;</span>);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; SendTemplatedMail(<span style="color: blue">new</span> <span style="color: blue">string</span>[] { recipient }, templateName, args);</p> <p style="margin: 0px">}</p> <p style="margin: 0px">&nbsp;</p> <p style="margin: 0px"><span style="color: blue">public</span> <span style="color: blue">static</span> <span style="color: blue">void</span> SendTemplatedMail(<span style="color: blue">string</span>[] recipients, <span style="color: blue">string</span> templateName, <span style="color: blue">params</span> <span style="color: blue">object</span>[] args) {</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: green">// Validate arguments</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (recipients == <span style="color: blue">null</span>) <span style="color: blue">throw</span> <span style="color: blue">new</span> <span style="color: #2b91af">ArgumentNullException</span>(<span style="color: #a31515">&quot;recipients&quot;</span>);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (templateName == <span style="color: blue">null</span>) <span style="color: blue">throw</span> <span style="color: blue">new</span> <span style="color: #2b91af">ArgumentNullException</span>(<span style="color: #a31515">&quot;templateName&quot;</span>);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (<span style="color: blue">string</span>.IsNullOrEmpty(templateName)) <span style="color: blue">throw</span> <span style="color: blue">new</span> <span style="color: #2b91af">ArgumentException</span>(<span style="color: #a31515">&quot;Value cannot be null or empty string.&quot;</span>, <span style="color: #a31515">&quot;templateName&quot;</span>);</p> <p style="margin: 0px">&nbsp;</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: green">// Get subject and body of message</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">string</span> subject = Resources.<span style="color: #2b91af">MailMessages</span>.ResourceManager.GetString(templateName + <span style="color: #a31515">&quot;Subject&quot;</span>);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: blue">string</span> body = Resources.<span style="color: #2b91af">MailMessages</span>.ResourceManager.GetString(templateName + <span style="color: #a31515">&quot;Body&quot;</span>);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; subject = <span style="color: blue">string</span>.Format(subject, args);</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; body = <span style="color: blue">string</span>.Format(body, args);</p> <p style="margin: 0px">&nbsp;</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; <span style="color: green">// Send messages</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; SendMail(recipients, subject, body);</p> <p style="margin: 0px">}</p></div> <p>I tato metoda má dva overloady, pro jedoho a pro více příjemců. Kromě adresáta vyžaduje ještě název šablony, který se potom použije pro určení odpovídajícího názvu resource položky. Dále pak obsahuje pole parametrů <code>args</code>, kam se předají argumenty, které metoda <code>string.Format</code> použije na místa zástupek <code>{0}</code>, <code>{1}</code> atd.</p> <p>Vlastní odeslání mailu tak lze provést příkladně takto:</p> <div style="font-size: 12pt; background: white; color: black; font-family: consolas, courier new"> <p style="margin: 0px"><span style="color: #2b91af">Mailer</span>.SendTemplatedMail(<span style="color: #a31515">&quot;someone@example.org&quot;</span>, <span style="color: #a31515">&quot;ApproveRequest&quot;</span>,</p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; (<span style="color: blue">int</span>)user.ProviderUserKey,&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: green">// 0</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; r.Nick,&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: green">// 1</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; r.Species,&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: green">// 2</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; r.Email,&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: green">// 3</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; r.FullName,&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: green">// 4</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; r.Citizenship,&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: green">// 5</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; r.Role&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: green">// 6</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; r.UiCulture,&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: green">// 7</span></p> <p style="margin: 0px">&nbsp;&nbsp;&nbsp; Altairis.Web.<span style="color: #2b91af">Utils</span>.BaseUrl()); <span style="color: green">// 8</span></p></div> <h3>Lokalizace</h3> <p>Pokud budeme chtít texty lokalizovat, stačí vytvořit kopii souboru <code>MailMessages.resx</code>, která bude před příponou obsahovat ISO kód jazyka, ve kterém texty jsou. Pro angličtinu se tedy soubor bude jmenovat <code>MailMessages.en.resx</code>. Pak přeložíme obsah souboru do požadovaného jazyka.</p> <p>RESX soubory jsou normální XML, pro jejich překlad tedy stačí běžný textový editor, překladatel nemusí mít Visual Studio. Pohodlnější ale bude použít k překladu specializovaný program. Doporučuji například <a href="http://www.peoplewords.com/download/ResxEditor.aspx">RESX Editor</a>, který je k dispozici zdarma.</p>