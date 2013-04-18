# Fakturownia API


Opis jak zintegrować własną aplikację lub serwis z systemem <http://fakturownia.pl/>


Dzięki API można z innych systemów wystawiać faktury/rachunki/paragony oraz zarządzać tymi dokumentami, a także klientami i produktami

## Spis treści
+ [API Token](#token)  
+ [Przykłady wywołania](#examples)  
+ [Link do podglądu faktury i pobieranie do PDF](#view_url)  
+ [Przykłady użycia  - zakup szkolenia](#use_case1)  
+ [Faktury - specyfikacja](#invoices)
+ [Przykłady w PHP i Ruby](#codes)  


<a name="token"/>
##API token

`API_TOKEN` token trzeba pobrać z ustawień aplikacji ("Ustawienia -> Ustawienia konta -> Integracja -> Kod autoryzacyjny API")

<a name="examples"/>
##Przykłady wywołania

Pobranie listy faktur z aktualnego miesiąca

```shell
curl https://twojaDomena.fakturownia.pl/invoices.json?period=this_month&api_token=API_TOKEN
```

<b>UWAGA</b>: do wywołań można przekazywać dodatkowe parametry - te same które są używane w aplikacji, np. `page=`, `period=` itp.

Faktury danego klienta

```shell
curl https://twojaDomena.fakturownia.pl/invoices.json?client_id=ID_KLIENTA&api_token=API_TOKEN
```

Pobranie faktury po ID


```shell
curl https://twojaDomena.fakturownia.pl/invoices/100.json?api_token=API_TOKEN
```

Pobranie PDF-a


```shell
curl https://twojaDomena.fakturownia.pl/invoices/100.pdf?api_token=API_TOKEN
```

inne opcje PDF:
* print_option=original - Oryginał
* print_option=copy - Kopia
* print_option=original_and_copy - Oryginał i kopia
* print_option=duplicate Duplikat


Dodanie nowej faktury

```shell
curl https://YOUR_DOMAIN.fakturownia.pl/invoices.json 
  	-H 'Accept: application/json'  
	-H 'Content-Type: application/json'  
	-d '{
	  	"api_token": "API_TOKEN",
	  	"invoice": {
			"kind":"vat", 
			"number": null, 
			"sell_date": "2013-01-16", 
			"issue_date": "2013-01-16", 
			"payment_to": "2013-01-23",
			"seller_name": "Wystawca Sp. z o.o.", 
			"seller_tax_no": "5252445767", 
			"buyer_name": "Klient1 Sp. z o.o.",
			"buyer_tax_no": "5252445767",
			"positions":[
				{"name":"Produkt A1", "tax":23, "total_price_gross":10.23, "quantity":1},
				{"name":"Produkt A2", "tax":0, "total_price_gross":50, "quantity":3}
			]		
		}
	}'
```

Aktualizacja faktury

```shell
curl https://YOUR_DOMAIN.fakturownia.pl/invoices/111.json 
	-X PUT 
	-H 'Accept: application/json'  
	-H 'Content-Type: application/json'  
	-d '{
		"api_tokn": "API_TOKEN",
		"invoice": {
			"buyer_name": "Nowa nazwa klienta Sp. z o.o."
		}
	}'
```

<a name="view_url"/>
##Link do podglądu faktury i pobieranie do PDF

Po pobraniu danych faktury np. przez:

```shell
curl https://twojaDomena.fakturownia.pl/invoices/100.json?api_token=API_TOKEN
```

API zwraca nam m.in. pole `token` na podstawie którego możemy otrzymać linki do podglądu faktury oraz zo pobrania PDF-a z wygenrowaną fakturą.
Linki takie umożliwiają odwołanie się do wybranej faktury  bez konieczności logowania - czyli możemy np. te linki przesłać klientowi, który otrzyma dostęp do faktury i PDF-a.

Lini te są postaci: 

podgląd: `http://twojaDomena.fakturownia.pl/invoice/{{token}}` 
pdf: `http://twojaDomena.fakturownia.pl/invoice/{{token}}.pdf`

Np dla tokenu równego: `HBO3Npx2OzSW79RQL7XV2` publiczny PDF będzie pod adresem `http://twojaDomena.fakturownia.pl/invoice/HBO3Npx2OzSW79RQL7XV2.pdf`

<a name="use_case1"/>
##Przykłady użycia w PHP - zakup szkolenia

`TODO` 

Przykład flow Portalu, który generuje dla klienta fakturę Proformę, wysyła ją klientowi i po opłaceniu wysyła do klienta bilet na szkolenie

* Klient wypełnia dane w Portalu
* Portal wywołuje API z fakturownia.pl i tworzy fakturę
* Portal pobiera wysyła Klientowi fakturę Proforma w PDF wraz z linkiem do płatności
* Klient opłaca fakturę Proforma (np. na PayPal lub PayU.pl)
* Fakturownia.pl otrzymuje informację, że płatność została wykonana, tworzy Fakturę VAT i wysyła ją Klientowi oraz wywołuje API Portalu
* Po otrzymaniu informacji o płatności (przez API) Portal wysyła Klientowi bilet na Szkolenie


<a name="invoices"/>
##Faktury


* `GET /invoices/1.json` pobranie faktury
* `POST /invoices.json` dodanie nowej faktury
* `PUT /invoices/1.json` aktualizacja faktury
* `DELETE /invoices/1.json` skasowanie faktury
 
Pola faktury

```shell
"number" : "13/2012" - number faktury (jeśli nie będzie podany wygeneruje się automatyczie)
"kind" : "vat" - rodzaj faktury (vat, proforma, bill, receipt, advance, correction, vat_mp, invoice_other, vat_margin, kp, kw, final, estimate)
"income" : "1" - fakturay przychodowa (1) lub kosztowa (0)
"issue_date" : "2013-01-16" - data wystawienia 
"place" : "Warszawa" - miejsce wystawienia
"sell_date" : "2013-01-16" - data sprzedaży (może być data lub miesiąc postaci 2012-12)
"category_id" : "" - id kategorii
"department_id" : "1" - id działu firmy
"seller_name" : "Radgost Sp. z o.o." - sprzedawca
"seller_tax_no" : "525-244-57-67" - nip sprzedawcy
"seller_bank_account" : "24 1140 1977 0000 5921 7200 1001" - konto bankowe sprzedawcy
"seller_bank" : "BRE Bank", 
"seller_post_code" : "02-548", 
"seller_city" : "Warszawa", 
"seller_street" : "ul. Olesińska 21", 
"seller_country" : "", 
"seller_email" : "platnosci@radgost123.com", 
"seller_www" : "", 
"seller_fax" : "", 
"seller_phone" : "", 
"client_id" : "-1" - id kupującego (jeśi -1 to klient zostanie utworzony w systemie)
"buyer_name" : "Nazwa klienta" - kupujący
"buyer_tax_no" : "525-244-57-67", 
"disable_tax_no_validation" : "", 
"buyer_post_code" : "30-314", 
"buyer_city" : "Warszawa", 
"buyer_street" : "Nowa 44", 
"buyer_country" : "", 
"buyer_note" : "", 
"buyer_email" : "", 
"additional_info" : "0" - czy wyświetlać dodatkowe pole na pozycjach faktury
"additional_info_desc" : "PKWiU" - nazwa dodatkowej kolumny na pozycjach faktury
"show_discount" : "0" - czy rabat
"payment_type" : "transfer", 
"payment_to_kind" : "other_date", 
"payment_to" : "2013-01-16", 
"status" : "issued", 
"paid" : "0,00", 
"oid" : "zamowienie10021", - numer zamówienia (np z zewnętrznego systemu zamówień)
"discount_kind" : "percent_unit", (dostępne wartości: 'percent_unit', 'percent_total', 'amount' )
"warehouse_id" : "1090", 
"seller_person" : "Imie Nazwisko", 
"buyer_first_name" : "Imie", 
"buyer_last_name" : "Nazwisko", 
"description" : "", 
"paid_date" : "", 
"currency" : "PLN", 
"lang" : "pl", 
"exchange_currency" : "", - przeliczona waluta (przeliczanie sumy i podatku na inną walutę, wg kursu NBP)
"internal_note" : "", 
"invoice_template_id" : "1", 
"description_footer" : "", 
"description_long" : "", 
"from_invoice_id" : "" - id faktury na podstawie której faktura została wygenerowana (przydatne np. w przypadku generacji Faktura VAT z Faktury Proforma)
"positions":
   		"product_id" : "1", 
   		"name" : "Fakturownia Start", 
   		"additional_info" : "", - dodatkowa informacja na pozycji faktury (np. PKWiU)
   		"discount_percent" : "", - zniżka procentowa (uwaga: aby rabat był wyliczany trzeba ustawić pole: 'show_discount' na '1')
   		"discount" : "", - zniżka kwotowa (uwaga: aby rabat był wyliczany trzeba ustawić pole: 'show_discount' na 1 oraz zmienić wartość pola 'discount_kind' na 'percent_total')
   		"quantity" : "1", 
   		"quantity_unit" : "szt", 
   		"price_net" : "59,00", - jeśli nie jest podana to zostanie wyliczona
   		"tax" : "23", 
   		"price_gross" : "72,57", - jeśli nie jest podana to zostanie wyliczona
   		"total_price_net" : "59,00", - jeśli nie jest podana to zostanie wyliczona
   		"total_price_gross" : "72,57"
```

Wartości pól

Pole: `kind`
```shell
	"vat" - faktura VAT
	"proforma" - faktura Proforma
	"bill" - rachunek
	"receipt" - paragon
	"advance" - faktura zaliczkowa
	"final" - faktura końcowa
	"correction" - faktura korekta
	"vat_mp" - faktura MP 
	"invoice_other" - inna faktura 
	"vat_margin" - faktura marża
	"kp" - kasa przyjmie
	"kw" - kasa wyda
	"estimate" - Estimate
```

Pole: `lang`
```shell
	"pl" - faktura w języku polskim
	"en" - język angielski
	"de" - język niemiecki
	"fr" - język francuski
	"pl_en" - faktura polsko/angielska
	"pl_de" - faktura polsko/niemiecka
	"pl_fr" - faktura polsko/francuska
```


Pole: `income`
```shell
	"1" - fakura przychodwa
	"0" - faktura kosztowa
```

Pole: `payment_type`
```shell
	"transfer" - przelew
	"card" - karta płatnicza
	"cash" -  gotówka
	"dowolny_inny_wpis_tekstowy" 
```

Pole: `status`
```shell
	"issued" - wystawiona
	"sent" - wysłana
	"paid" - opłacona
	"partial" - częściowo opłacona
```

Pole: `discount_kind` - rodzaj rabatu
```shell
	"percent_unit" - liczony od ceny jednostkowej
	"percent_total" - liczony od ceny całkowitej
	"amount" - kwotowy
```


<a name="codes"/>
##Przykłady w PHP i Ruby
