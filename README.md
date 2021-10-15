# 2023-e-Dogru-Turkiye-nin-Buyuyen-Firmalari
*** 04.07.2021 ***
***** 2023'e Doğru Türkiyen'nin Büyüyen Firmaları ****
clear all
clear mata
set more off, permanently
set niceness 3, permanently
cap log close
cd "xxx\20210407_Hızlı Büyüyen Firmalar"

********************************************************************************
* İmalat ve Bilgi İletişim Sektörleri cvs *
import delimited "xxx\20210407_Hızlı Büyüyen Firmalar\xxxx.csv", encoding(ISO-8859-9) 
describe
destring çalışansayısıtoplamücret ithalatdeğer ihracatdeğer kosgebdestek tübitakdestek, replace dpcomma
save "xxx\20210407_Hızlı Büyüyen Firmalar\Yeni klasör\xxxx.dta"
clear all

* Bilanço cvs*
import delimited "xxx\20210407_Hızlı Büyüyen Firmalar\xxxx.csv", encoding(ISO-8859-9) 
describe
destring *, replace dpcomma
order ID donem sırano_donem donem_yıl ıdönenvarlıklar, b( ahazırdeğerler )
save "xxx\20210407_Hızlı Büyüyen Firmalar\Yeni klasör\xxxx.dta"
clear all
********************************************************************************,
* Merge *
* Bilanço *
use "xxx\20210407_Hızlı Büyüyen Firmalar\Yeni klasör\xxxx.dta"
sort ID donem_yıl
drop donem
label variable ID "ID"
label variable donem_yıl "dönem"
rename donem_yıl dönem
save "xxx\20210407_Hızlı Büyüyen Firmalar\Yeni klasör\xxx.dta", replace
clear all

* Sektör *
sort ID fkdonemyıl
label variable ID "ID"
label variable fkdonemyıl "dönem"
rename fkdonemyıl dönem
save "xxx\20210407_Hızlı Büyüyen Firmalar\Yeni klasör\xxx.dta", replace
clear all

********************************************************************************
*İsim düzeltmeleri*
*
*
*
*
*

*
*
*
*
*
*
********************************************************************************
*Duplicate Kontrol* 
duplicates tag ID dönem , gen(tagged)
tab tagged
bys ID dönem: gen sıra=_n if ID==ID & dönem==dönem
drop if tagged==3 & sıra>1

********** Calisan Sayısı *******
forval i=1/4 {
	gen X`i'=1 if calisansayisi_`i'_ceyrek==.
	replace X`i'=0 if X`i'==.
}
*******************************************
cap drop temp* 	
gen temp=X1+X2+X3+X4
egen calisansayisi=rowmean(calisansayisi_1_ceyrek calisansayisi_2_ceyrek calisansayisi_3_ceyrek calisansayisi_4_ceyrek)
replace calisansayisi=. if temp>3
replace calisansayisi=int(calisansayisi) if calisansayisi>=1
sum calisansayisi, d

******* Toplam Ücret ***********
forval i=1/4 {
	gen Y`i'=1 if calisansayisi_`i'_ceyrek_ucret==.
	replace Y`i'=0 if Y`i'==.
} 
*****************************************
cap drop temp* 
gen temp_boşluk=Y1+Y2+Y3+Y4
gen temp1= 3* calisansayisi_1_ceyrek_ucret/*maaşın 3 aya yayılması*/
gen temp2= 3* calisansayisi_2_ceyrek_ucret 
gen temp3= 3* calisansayisi_3_ceyrek_ucret
gen temp4= 3* calisansayisi_4_ceyrek_ucret
egen toplamucret=rowmean(temp1 temp2 temp3 temp4)
replace toplamucret=. if temp_boşluk>3
cap drop temp* 

** Enflasyon

merge m:m dönem nace2_2dig using "xxx\Desktop\Yİ_ÜFE 2003.dta"
drop if _m<3
bys ID dönem: gen çalışan_başına_ücret=toplamucret/calisansayisi
bys ID nace2_2dig (dönem): gen def_çalışan_ücret=(çalışan_başına_ücret/Yİ_ÜFE)*100


******** Değer Etiketleme **************
replace teknolojiduzeyi2aciklama="Bilgi İletişim" if teknolojiduzeyi2aciklama=="Belirsiz"

label define KOBİ_tanım 1 "Mikro Ölçekli İşletme" 2 "Küçük Ölçekli İşletme" 3 "Orta Ölçekli İşletme" 4 "KOBİ Üstü"
label values kobi_2018 KOBİ_tanım

label define Mükellef_Türü 1 "Kurumlar Vergisine Tabi Olanlar" 2 "Gelir Beyannamesine Tabi Olanlar"
label values mukellefturu Mükellef_Türü

gen kosgeb_destek_alan=1 if kosgebdestek>0 & kosgebdestek!=.
replace kosgeb_destek_alan=0 if kosgeb_destek_alan==.
br if kosgebdestek==0
label variable kosgeb_destek_alan "Kosgeb destek alıp almaması"
label define kosgeb_destek 1 "Kosgeb desteği alan" 0 "diğerleri"
label values kosgeb_destek_alan kosgeb_destek

gen tübitak_destek_alan=1 if tübitakdestek>0 &tübitakdestek!=.
replace tübitak_destek_alan=0 if tübitak_destek_alan==.
label variable tübitak_destek_alan "Tübitak destek alıp almaması"
label define tübitak_destek 1 "Tübitak desteği alan" 0 "diğerleri"
label values tübitak_destek_alan tübitak_destek

gen patent_alan_girişim= 1 if patentbaşvurusayısı>0 & patentbaşvurusayısı!=.
replace patent_alan_girişim=0 if patent_alan_girişim==.
label variable patent_alan_girişim "Patent Alan Girişim "
label define patent_başvurusunda_bulunan 1 "Patent Başavurusunda Bulunma" 0 "diğerleri"
label values patent_alan_girişim patent_başvurusunda_bulunan
order patent_alan_girişim, a( patentbaşvurusayısı)

gen tasarım_başvuru_girişim= 1 if tasarımbaşvurusayısı>0 & tasarımbaşvurusayısı!=.
replace tasarım_başvuru_girişim=0 if tasarım_başvuru_girişim==.
label variable tasarım_başvuru_girişim "Tasarım Yapan Girişim "
label define tasarım_başvurusunda_bulunan 1 "Tasarım Başavurusunda Bulunma" 0 "diğerleri"
label values tasarım_başvuru_girişim tasarım_başvurusunda_bulunan
order tasarım_başvuru_girişim, a(tasarımbaşvurusayısı)

gen marka_başvuru_girişim=1 if markabaşvurusayısı>0 & markabaşvurusayısı!=.
replace marka_başvuru_girişim=0 if marka_başvuru_girişim==.
label variable marka_başvuru_girişim "Marka Başvuru Girşimi"
label define marka_başvuru_girişim_ 1 "Marka Başvurusunda Bulunanlar" 0 "Diğerleri"
label values marka_başvuru_girişim marka_başvuru_girişim_
order marka_başvuru_girişim, a(markabaşvurusayısı)

********************************************************************************
* Nace Rev.2 Düzenleme
split nacesinifaciklama, p(-)
tab nacesinifaciklama3
drop nacesinifaciklama2 nacesinifaciklama3
destring nacesinifaciklama1, replace
rename nacesinifaciklama1 nace2_4dig
bys ID (dönem): gen nace2_2dig=floor(nace2_4dig/100)

********************************************************************************
* İşe başlama yılı 
bys ID: gen başlangıc=işe_başlama 
bys ID: replace başlangıc=kurum_yılı if başlangıc==.
bys ID: gen yas=dönem-başlangıc


* Açılan ve Kapanan Girişimler
*Girişim Sayısı
bys ID (dönem): gen one=1
bys dönem: egen imalat_girişim_sayısı=sum(one) if nacekisimaciklama=="C - İMALAT"
bys dönem: egen bilişim_girişim_sayısı=sum(one) if nacekisimaciklama=="J - BİLGİ VE İLETİŞİM"

preserve
collapse (mean) imalat_girişim_sayısı bilişim_girişim_sayısı, by(dönem)
export excel using "xxx\20210407_Hızlı Büyüyen Firmalar\Çıktılar\xxx.xlsx", sheet("1") sheetmodify firstrow(variables)
restore

bys dönem nace2_2dig: egen sektörel_girişim_sayısı=sum(one) 
preserve
collapse (mean) sektörel_girişim_sayısı, by(dönem nace2_2dig)
export excel using "xxx\20210407_Hızlı Büyüyen Firmalar\Çıktılar\xxx.xlsx", sheet("2") sheetmodify firstrow(variables)
restore

* Açılan
bys ID (dönem): egen ilkyil=min(dönem)
bys ID (dönem): egen sonyil=max(dönem)

bys ID (dönem): gen period=_n /*girişim kaç dönemlik bilgisi yer alıyor */
bys ID (dönem): egen duration=max(period) /* max donem sayısı*/

gen entry=1 if yas==0 &ilkyil>=2006 & dönem!=2006
replace entry=1 if yas==0 & dönem==2006
replace entry=0 if entry==.
label variable entry "Yeni_kurulan"
label define Yeni_kurulan 1 "Kurulan İş Yeri" 0 "Diğer"
label values entry Yeni_kurulan

cap drop break
bys ID (dönem): gen break=1 if ID==ID[_n+1] &  ( dönem!=dönem[_n+1]-1)

*Kapanan 

bys ID (dönem): gen exit=1 if ID!=ID[_n+1] & break!=1 & sonyil==dönem & dönem!=2019
replace exit=0 if exit==.
label define Çıkış_girişim 0 "diğer" 1 "Çıkış yapan girişim"
label variable exit "Çıkış_girişim"
label values exit Çıkış_girişim

********************************************************************************
* Bölge ve Sektör Bilgileri 
cap drop _m
merge m:1 plaka using "xxx\Desktop\il_nuts2_lookup.dta"
cap drop _m

gen nuts2no=1 if nuts2=="TR10"
replace nuts2no=2 if nuts2=="TR21"
replace nuts2no=3 if nuts2=="TR22"
replace nuts2no=4 if nuts2=="TR31"
replace nuts2no=5 if nuts2=="TR32"
replace nuts2no=6 if nuts2=="TR33"
replace nuts2no=7 if nuts2=="TR41"
replace nuts2no=8 if nuts2=="TR42"
replace nuts2no=9 if nuts2=="TR51"
replace nuts2no=10 if nuts2=="TR52"
replace nuts2no=11 if nuts2=="TR61"
replace nuts2no=12 if nuts2=="TR62"
replace nuts2no=13 if nuts2=="TR63"
replace nuts2no=14 if nuts2=="TR71"
replace nuts2no=15 if nuts2=="TR72"
replace nuts2no=16 if nuts2=="TR81"
replace nuts2no=17 if nuts2=="TR82"
replace nuts2no=18 if nuts2=="TR83"
replace nuts2no=19 if nuts2=="TR90"
replace nuts2no=20 if nuts2=="TRA1"
replace nuts2no=21 if nuts2=="TRA2"
replace nuts2no=22 if nuts2=="TRB1"
replace nuts2no=23 if nuts2=="TRB2"
replace nuts2no=24 if nuts2=="TRC1"
replace nuts2no=25 if nuts2=="TRC2"
replace nuts2no=26 if nuts2=="TRC3"

label define nuts2no_label 1 "TR10- İstanbul" 2 "TR21-Tekirdağ, Edirne, Kırklareli" 3 "TR22-Balıkesir, Çanakkale" 4 "TR31-İzmir" 5 "TR32-Aydın, Denizli, Muğla" 6 "TR33-Manisa, Afyonkarahisar, Kütahya, Uşak" 7 "TR41-Bursa, Eskişehir, Bilecik" 8 "TR42-Kocaeli, Sakarya, Düzce, Bolu, Yalova" 9 "TR51-Ankara" 10 "TR52-Konya, Karaman" 11 "TR61-Antalya, Isparta, Burdur" 12 "TR62-Adana, Mersin" 13 "TR63-Hatay, Kahramanmaraş, Osmaniye" 14 "TR71-Kırıkkale, Aksaray, Niğde, Nevşehir, Kırşehir" 15 "TR72-Kayseri, Sivas, Yozgat" 16 "TR81-Zonguldak, Karabük, Bartın" 17 "TR82-Kastamonu, Çankırı, Sinop" 18 "TR83-Samsun, Tokat, Çorum, Amasya" 19 "TR90-Trabzon, Ordu, Giresun, Rize, Artvin, Gümüşhane" 20 "TRA1-Erzurum, Erzincan, Bayburt" 21 "TRA2-Ağrı, Kars, Iğdır, Ardahan" 22 "TRB1-Malatya, Elazığ, Bingöl, Tunceli" 23 "TRB2-Van, Muş, Bitlis, Hakkari" 24 "TRC1-Gaziantep, Adıyaman, Kilis" 25 "TRC2-Şanlıurfa, Diyarbakır" 26 "TRC3-Mardin, Batman, Şırnak, Siirt"

label variable nuts2no "Nuts2 bölge numarası ve il adları"
label values nuts2no nuts2no_label

tab kobi_2018 if yas>=0  & yas<=1 & nuts2==""
tab dönem kobi_2018 if yas>=0  & yas<=1 & nuts2==""
drop if yas>=0  & yas<=1 & nuts2=="" &kobi_2018==1

gen temp=1 if plaka==99
replace plaka=34 if temp==1
replace iladı="İstanbul" if temp==1 
* dikkatli yap *
/*
replace nuts2no=1 if temp==1
replace iladı = "Elazığ" in 60629
replace iladı = "Çorum" in 110411
replace iladı = "Afyonkarahisar" in 278921
*/
cap drop temp
********************************************************************************
*Örneklem Oluşturma
* Mikro Kısıtı
sort ID dönem
bys ID: gen mikro=1 if kobi_2018==1
replace mikro=0 if mikro==.

sort ID dönem
bys ID : egen top_mikro=sum(mikro)
bys ID: gen mikro_kısıtı=0  if top_mikro==duration
replace mikro_kısıtı=1 if top_mikro!=duration
tab kobi_2018  mikro_kısıtı

*Dönem Kısıtları
sort ID dönem
cap drop one
bys ID dönem: gen one_20062009=1 if dönem>=2006 &dönem<2010
replace one_20062009=0 if one_20062009==.
bys ID: egen sum_20062009=sum(one_20062009) if one_20062009==1

cap drop one
bys ID dönem: gen one_20072010=1 if dönem>=2007 &dönem<2011
replace one_20072010=0 if one_20072010==.
bys ID: egen sum_20072010=sum(one_20072010) if one_20072010==1

cap drop one
bys ID dönem: gen one_20082011=1 if dönem>=2008 &dönem<2012
replace one_20082011=0 if one_20082011==.
bys ID: egen sum_20082011=sum(one_20082011) if one_20082011==1

cap drop one
bys ID dönem: gen one_20092012=1 if dönem>=2009 &dönem<2013
replace one_20092012=0 if one_20092012==.
bys ID: egen sum_20092012=sum(one_20092012) if one_20092012==1

cap drop one
bys ID dönem: gen one_20102013=1 if dönem>=2010 &dönem<2014
replace one_20102013=0 if one_20102013==.
bys ID: egen sum_20102013=sum(one_20102013) if one_20102013==1

cap drop one
bys ID dönem: gen one_20112014=1 if dönem>=2011 &dönem<2015
replace one_20112014=0 if one_20112014==.
bys ID: egen sum_20112014=sum(one_20112014) if  one_20112014==1

cap drop one
bys ID dönem: gen one_20122015=1 if dönem>=2012 &dönem<2016
replace one_20122015=0 if one_20122015==.
bys ID: egen sum_20122015=sum(one_20122015) if one_20122015==1

cap drop one
bys ID dönem: gen one_20132016=1 if dönem>=2013 &dönem<2017
replace one_20132016=0 if one_20132016==.
bys ID: egen sum_20132016=sum(one_20132016) if one_20132016==1

cap drop one
bys ID dönem: gen one_20142017=1 if dönem>=2014 &dönem<2018
replace one_20142017=0 if one_20142017==.
bys ID: egen sum_20142017=sum(one_20142017) if one_20142017==1 

cap drop one
bys ID dönem: gen one_20152018=1 if dönem>=2015 &dönem<2019
replace one_20152018=0 if one_20152018==.
bys ID: egen sum_20152018=sum(one_20152018) if one_20152018==1

cap drop one
bys ID dönem:gen one_20162019=1 if dönem>=2016 &dönem<2020
replace one_20162019=0 if one_20162019==.
bys ID: egen sum_20162019=sum(one_20162019) if one_20162019==1

cap drop one

**küçük ölçekli işletme olup dönemler içinde mikroya düşenlerin araştırması **

* Örneklem
gen örneklem=1 if mikro_kısıtı==1
replace örneklem=0 if örneklem==.
keep if örneklem==1

* Firmaların Net Satışlarının Reelleştirmesi 
order cnetsatışlar , a( örneklem )
sort ID dönem
bys ID (dönem): gen def_net_satış=(cnetsatışlar/Yİ_ÜFE)*100

*outlier
bys dönem: egen eşik1=pctile(def_net_satış) if def_net_satış!=., p(0.001)
bys dönem: egen eşik2=pctile(def_net_satış) if def_net_satış!=., p(99.999)

***** Net satış Büyüme ****
forval i=2006/2019 {
	sort ID dönem
	bys ID:gen r_cnetsatislar`i'=def_net_satış if  dönem==`i'
}

forval i=2006/2019 {
	sort ID dönem
	bys ID: fillmissing r_cnetsatislar`i' , with(previous)
	bys ID: fillmissing r_cnetsatislar`i' , with(next)
}


*******************************************************************************
bys ID: gen büyüme2006_2009=((r_cnetsatislar2009/r_cnetsatislar2006)^(1/4))-1 
bys ID: replace büyüme2006_2009=büyüme2006_2009*100

bys ID: gen büyüme2007_2010=((r_cnetsatislar2010/r_cnetsatislar2007)^(1/4))-1 
bys ID: replace büyüme2007_2010=büyüme2007_2010*100

bys ID: gen büyüme2008_2011=((r_cnetsatislar2011/r_cnetsatislar2008)^(1/4))-1 
bys ID: replace büyüme2008_2011=büyüme2008_2011*100

bys ID: gen büyüme2009_2012=((r_cnetsatislar2012/r_cnetsatislar2009)^(1/4))-1 
bys ID: replace büyüme2009_2012=büyüme2009_2012*100

bys ID: gen büyüme2010_2013=((r_cnetsatislar2013/r_cnetsatislar2010)^(1/4))-1 
bys ID: replace büyüme2010_2013=büyüme2010_2013*100

bys ID: gen büyüme2011_2014=((r_cnetsatislar2014/r_cnetsatislar2011)^(1/4))-1 
bys ID: replace büyüme2011_2014=büyüme2011_2014*100

bys ID: gen büyüme2012_2015=((r_cnetsatislar2015/r_cnetsatislar2012)^(1/4))-1 
bys ID: replace büyüme2012_2015=büyüme2012_2015*100

bys ID: gen büyüme2013_2016=((r_cnetsatislar2016/r_cnetsatislar2013)^(1/4))-1 
bys ID: replace büyüme2013_2016=büyüme2013_2016*100

bys ID: gen büyüme2014_2017=((r_cnetsatislar2017/r_cnetsatislar2014)^(1/4))-1
bys ID: replace büyüme2014_2017=büyüme2014_2017*100

bys ID: gen büyüme2015_2018=((r_cnetsatislar2018/r_cnetsatislar2015)^(1/4))-1
bys ID: replace büyüme2015_2018=büyüme2015_2018*100

bys ID: gen büyüme2016_2019=((r_cnetsatislar2019/r_cnetsatislar2016)^(1/4))-1
bys ID: replace büyüme2016_2019=büyüme2016_2019*100

rename büyüme2006_2009 büyüme2006
rename büyüme2007_2010 büyüme2007
rename büyüme2008_2011 büyüme2008
rename büyüme2009_2012 büyüme2009
rename büyüme2010_2013 büyüme2010
rename büyüme2011_2014 büyüme2011
rename büyüme2012_2015 büyüme2012
rename büyüme2013_2016 büyüme2013
rename büyüme2014_2017 büyüme2014
rename büyüme2015_2018 büyüme2015
rename büyüme2016_2019 büyüme2016

forval i=2006/2016{
	bys ID: gen büyüme`i'_açıklama=1 if büyüme`i'>=20 & büyüme`i'!=. & exit==0 
	bys ID: replace büyüme`i'_açıklama=2 if büyüme`i'<20 & büyüme`i'>=10 &  büyüme`i'!=. & exit==0 
	bys ID: replace büyüme`i'_açıklama=3 if büyüme`i'<10 & büyüme`i'>=0 & büyüme`i'!=. & exit==0 
	bys ID: replace büyüme`i'_açıklama=4 if büyüme`i'<0 & büyüme`i'>=-10 & büyüme`i'!=. & exit==0 
	bys ID: replace büyüme`i'_açıklama=5 if büyüme`i'<-10 & büyüme`i'>=-20 & büyüme`i'!=. & exit==0 
	bys ID: replace büyüme`i'_açıklama=6 if büyüme`i'<-20 & büyüme`i'!=. & exit==0 
	cap label drop açıklama`i'
	label define açıklama`i' 1 "hızlı büyüyen" 2 "hızlı büyüme potansiyeli olan" 3 "yavaş büyüyen" 4 "yavaş küçülen" 5 "hızlı küçülme potansiyeli olan" 6 "hızlı küçülen"
	label values büyüme`i'_açıklama açıklama`i'
}


**** Çalısan Sayısında Büyüme ***
forval i=2006/2019 {
	sort ID dönem
	bys ID:gen çalışansayısı`i'=calisansayisi if  dönem==`i'
}

forval i=2006/2019 {
	sort ID dönem
	bys ID: fillmissing çalışansayısı`i' , with(previous)
	bys ID: fillmissing çalışansayısı`i' , with(next)
}

*******************************************************************************
bys ID: gen çalışan_büyüme2006=((çalışansayısı2009/çalışansayısı2006)^(1/4))-1 
bys ID: replace çalışan_büyüme2006=çalışan_büyüme2006*100

bys ID: gen çalışan_büyüme2007=((çalışansayısı2010/çalışansayısı2007)^(1/4))-1 
bys ID: replace çalışan_büyüme2007=çalışan_büyüme2007*100

bys ID: gen çalışan_büyüme2008=((çalışansayısı2011/çalışansayısı2008)^(1/4))-1 
bys ID: replace çalışan_büyüme2008=çalışan_büyüme2008*100

bys ID: gen çalışan_büyüme2009=((çalışansayısı2012/çalışansayısı2009)^(1/4))-1 
bys ID: replace çalışan_büyüme2009=çalışan_büyüme2009*100

bys ID: gen çalışan_büyüme2010=((çalışansayısı2013/çalışansayısı2010)^(1/4))-1 
bys ID: replace çalışan_büyüme2010=çalışan_büyüme2010*100

bys ID: gen çalışan_büyüme2011=((çalışansayısı2014/çalışansayısı2011)^(1/4))-1 
bys ID: replace çalışan_büyüme2011=çalışan_büyüme2011*100

bys ID: gen çalışan_büyüme2012=((çalışansayısı2015/çalışansayısı2012)^(1/4))-1 
bys ID: replace çalışan_büyüme2012=çalışan_büyüme2012*100

bys ID: gen çalışan_büyüme2013=((çalışansayısı2016/çalışansayısı2013)^(1/4))-1 
bys ID: replace çalışan_büyüme2013=çalışan_büyüme2013*100

bys ID: gen çalışan_büyüme2014=((çalışansayısı2017/çalışansayısı2014)^(1/4))-1 
bys ID: replace çalışan_büyüme2014=çalışan_büyüme2014*100

bys ID: gen çalışan_büyüme2015=((çalışansayısı2018/çalışansayısı2015)^(1/4))-1 
bys ID: replace çalışan_büyüme2015=çalışan_büyüme2015*100

bys ID: gen çalışan_büyüme2016=((çalışansayısı2019/çalışansayısı2016)^(1/4))-1 
bys ID: replace çalışan_büyüme2016=çalışan_büyüme2016*100


forval i=2006/2016{
	bys ID: gen çalışan_büyüme`i'_açıklama=1 if çalışan_büyüme`i'>=20 & çalışan_büyüme`i'!=. & exit==0 
	bys ID: replace çalışan_büyüme`i'_açıklama=2 if çalışan_büyüme`i'<20 & çalışan_büyüme`i'>=10 &  çalışan_büyüme`i'!=. & exit==0 
	bys ID: replace çalışan_büyüme`i'_açıklama=3 if çalışan_büyüme`i'<10 & çalışan_büyüme`i'>=0 & çalışan_büyüme`i'!=. & exit==0 
	bys ID: replace çalışan_büyüme`i'_açıklama=4 if çalışan_büyüme`i'<0 & çalışan_büyüme`i'>=-10 & çalışan_büyüme`i'!=. & exit==0 
	bys ID: replace çalışan_büyüme`i'_açıklama=5 if çalışan_büyüme`i'<-10 & çalışan_büyüme`i'>=-20 & çalışan_büyüme`i'!=. & exit==0 
	bys ID: replace çalışan_büyüme`i'_açıklama=6 if çalışan_büyüme`i'<-20 & çalışan_büyüme`i'!=. & exit==0 
	cap label drop çalışan_açıklama`i'
	label define çalışan_açıklama`i' 1 "hızlı büyüyen" 2 "hızlı büyüme potansiyeli olan" 3 "yavaş büyüyen" 4 "yavaş küçülen" 5 "hızlı küçülme potansiyeli olan" 6 "hızlı küçülen"
	label values çalışan_büyüme`i'_açıklama çalışan_açıklama`i'
}


*** Hızlı Büyüyen Firma Türleri ***
*** Atlar
forval i=2006/2016{
	gen atlar`i'=1 if büyüme`i'>=20 & çalışan_büyüme`i'>=20 & büyüme`i'!=. & çalışan_büyüme`i'!=. 
	replace atlar`i'=0 if atlar`i'==.
	cap label drop atlar_açıklama`i'
	label define atlar_açıklama`i' 1 "Atlar" 0 "Diğerleri"
	label values atlar`i'  atlar_açıklama`i'
}
*** Tazılar
forval i=2006/2016{
	gen tazılar`i'=1 if büyüme`i'>=20 & büyüme`i'!=. 
	replace tazılar`i'=0 if tazılar`i'==.
	cap label drop tazılar_açıklama`i'
	label define tazılar_açıklama`i' 1 "Tazılar" 0 "Diğerleri"
	label values tazılar`i' tazılar_açıklama`i'

}	
*** Karıncalar
forval i=2006/2016{
	gen karıncalar`i'=1 if çalışan_büyüme`i'>=20 & çalışan_büyüme`i'!=. 
	replace karıncalar`i'=0 if karıncalar`i'==.
	cap label drop karıncalar_açıklama`i'
	label define karıncalar_açıklama`i' 1 "Karıncalar" 0 "Diğerleri"
	label values karıncalar`i' karıncalar_açıklama`i'

}	

*** Ceylanlar
forval i=2006/2016{
	gen ceylanlar`i'=1 if (büyüme`i'>=20 | çalışan_büyüme`i'>=20) & büyüme`i'!=. & çalışan_büyüme`i'!=. & yas>5 
	replace ceylanlar`i'=0 if ceylanlar`i'==.
	cap label drop ceylanlar_açıklama`i'
	label define ceylanlar_açıklama`i' 1 "Ceylanlar" 0 "Diğerleri"
	label values ceylanlar`i' ceylanlar_açıklama`i'

}	

*** Gazeller
forval i=2006/2016{
	gen gazeller`i'=1 if (büyüme`i'>=20 | çalışan_büyüme`i'>=20) & büyüme`i'!=. & çalışan_büyüme`i'!=. & yas<=5  
	replace gazeller`i'=0 if gazeller`i'==.
	cap label drop gazeller_açıklama`i'
	label define gazeller_açıklama`i' 1 "Gazeller" 0 "Diğerleri"
	label values gazeller`i' gazeller_açıklama`i'
}	



forval i=2006/2016{
	count if büyüme`i'_açıklama==1 & dönem==`i'+3
}
******

forval i=2006/2016{
	count if çalışan_büyüme`i'_açıklama==1 & dönem==`i'+3
}
******

forval i=2006/2016{
	count if atlar`i'==1 & dönem==`i'+3
}
******

forval i=2006/2016{
	count if tazılar`i'==1 & dönem==`i'+3
}
******

forval i=2006/2016{
	count if karıncalar`i'==1 & dönem==`i'+3
}
******

forval i=2006/2016{
	count if ceylanlar`i'==1 & dönem==`i'+3
}
******

forval i=2006/2016{
	count if gazeller`i'==1 & dönem==`i'+3
}




* 05.10.2021-çalışma bitti *






