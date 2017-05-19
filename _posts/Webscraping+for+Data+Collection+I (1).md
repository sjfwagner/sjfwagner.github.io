
# Webscraping for Data Collection

I ~love~ using webscraping to grab data from content on websites. In my day job this is something I end up using a lot, getting data from different websites, putting it back together, and it's also something I get a lot of questions about. Webscraping is particularily useful when 1) the service is not intended to provide data or 2) there is no good public api. This simple tutorial will build into my "Women's Representation in History" as I show how to webscrape data from webpages at different levels of diffculty. 

I'm not an expert in this! There is always a better way to solve a problem, but here are some methods that are pretty quick that should get you going, and also get you on the path to scraping your own data from webpages.

I plan to do at least 3 tutorials with different levels of capabilities. 

1. Very simple (this one!) basic request and parsing of html
    1. How to follow a link, and how to recursively scrape
3. Creating a spider, doing crawls, sending scraped data back to a data base
4. Handling javascript rendered pages (and looking for "hidden" apis)

### Basic page request and html parsing
Most of the time I use `lxml` and `requests` for light webscraping. There are other libraries, but I find that being conversant in these two are usually enough to solve more that 90% of my webscraping needs (you can see how some of my data collection from pt1 of my Women's Representation in History was done using this library [here](https://github.com/sjfwagner/Blog-code-snippets-and-notebooks/blob/master/SYMIH_Scrapes.ipynb)). Preferences may vary! [BeautifulSoup4](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) is very popular. There *are* other HTTP request libraries, like urllib2 etc, but I have never seen a HTTP requests library recommended over requests. 


```python
#the two imports that we'll need immediately
import requests
from lxml import html
```

For this simple tutorial we'll be scraping a page from wikipedia, on historians in history. 
You can see the page here, and I also recommend that you view the source, as that will give you a good idea of the structure of the page we'll be scraping. Note! There are specific packages for wikipedia -- so if you want you should use them, this is an example of how to get started on scraping.

We'll start [here](https://en.wikipedia.org/wiki/List_of_historians), with a list of historians identified by wikipedia and then use the same techniques to move on to [here](https://en.wikipedia.org/wiki/List_of_historians_by_area_of_study), a list of historians identified by wikipedia for each area of study. There will be overlap, but I believe each list may have some different names. 


```python
url = 'https://en.wikipedia.org/wiki/List_of_historians'
response = requests.get(url)
```

Generically html is just a structured document, whose tags allow the browser to figure out how to display information. CSS and Javascript modify this, but most webpages present simple html. 

We can use the structure of the document, the tags, the pull features out of this. Xpath is a structured language for querying xml documents (great [tutorial](http://www.w3schools.com/xsl/xpath_intro.asp) at w3 schools) this allows us to easily query the document for the information that we want, because, lucky for us, html is a type of xml document. Xpath looks at the tags (like "div" etc) and the associated properties to reference different bits of the relevant html. 

It's super useful at this point to load the source (right-click view source) get a generic idea of the kind of html you'll be combing through. Keep this up throughout to use as a reference. If you'd like you can also pull up your browser's developer tools from the settings, many of which will highlight the section of html you're in while your mouse moves around the page. They will also provide the exact Xpath to the portion of the source you have highlighted *but this is generally way too specific, and will not extend well to many pages*


```python
wiki_page = html.fromstring(response.content)
```

Quickly, the xpath features I find it most useful to know are these: 
- "//" no matter the structure match to this tag 
    -i.e. "//div" will match every div tag
- xpath queries build, so "//div//li" will find *any* list tag **inside** *any* div tag
    - //div/li will return list items just inside a div tag
- [@....] the @ allows you to grab information from the tag itself
    -//div[@id="content"] will select any div tag *which* has a id of 'content'
    -//a[@title] will select all a tags with a title defined
There are other useful functions that you can use to more intelligently use xpath, but this will get you pretty far.



```python
historians = [] # create a json format of our data
for i in wiki_page.xpath('//div[@id="mw-content-text"]//li/a[@title][position()=1]'): #this xpath grabs all the list items, with a title
    print i.text #print the historian's name #unnecessary
    print i.xpath('@href') #print the link #unneccessary
    
    historians.append({"name":i.text, "url": "https://en.wikipedia.org"+i.xpath('@href')[0] })#store the full link
```

    Herodotus
    ['/wiki/Herodotus']
    Thucydides
    ['/wiki/Thucydides']
    Xenophon
    ['/wiki/Xenophon']
    Ctesias
    ['/wiki/Ctesias']
    Theopompus
    ['/wiki/Theopompus']
    Eudemus of Rhodes
    ['/wiki/Eudemus_of_Rhodes']
    Berossus
    ['/wiki/Berossus']
    Ptolemy I Soter
    ['/wiki/Ptolemy_I_Soter']
    Duris of Samos
    ['/wiki/Duris_of_Samos']
    Manetho
    ['/wiki/Manetho']
    Timaeus of Tauromenium
    ['/wiki/Timaeus_(historian)']
    Quintus Fabius Pictor
    ['/wiki/Quintus_Fabius_Pictor']
    Artapanus of Alexandria
    ['/wiki/Artapanus_of_Alexandria']
    Cato the Elder
    ['/wiki/Cato_the_Elder']
    Gaius Acilius
    ['/wiki/Gaius_Acilius']
    Polybius
    ['/wiki/Polybius']
    Sempronius Asellio
    ['/wiki/Sempronius_Asellio']
    Sima Tan
    ['/wiki/Sima_Tan']
    Sima Qian
    ['/wiki/Sima_Qian']
    Agatharchides
    ['/wiki/Agatharchides']
    Posidonius
    ['/wiki/Posidonius']
    Julius Caesar
    ['/wiki/Julius_Caesar']
    Diodorus of Sicily
    ['/wiki/Diodorus_Siculus']
    Sallust
    ['/wiki/Sallust']
    Liu Xiang (scholar)
    ['/wiki/Liu_Xiang_(scholar)']
    Theophanes of Mytilene
    ['/wiki/Theophanes_of_Mytilene']
    Dionysius of Halicarnassus
    ['/wiki/Dionysius_of_Halicarnassus']
    Strabo
    ['/wiki/Strabo']
    Livy
    ['/wiki/Livy']
    Marcus Velleius Paterculus
    ['/wiki/Marcus_Velleius_Paterculus']
    Memnon of Heraclea
    ['/wiki/Memnon_of_Heraclea']
    Ban Biao
    ['/wiki/Ban_Biao']
    Quintus Curtius Rufus
    ['/wiki/Quintus_Curtius_Rufus']
    Ban Gu
    ['/wiki/Ban_Gu']
    Flavius Josephus
    ['/wiki/Josephus']
    Pamphile of Epidaurus
    ['/wiki/Pamphile_of_Epidaurus']
    Ban Zhao
    ['/wiki/Ban_Zhao']
    Thallus
    ['/wiki/Thallus_(historian)']
    Plutarch
    ['/wiki/Plutarch']
    Gaius Cornelius Tacitus
    ['/wiki/Gaius_Cornelius_Tacitus']
    Suetonius
    ['/wiki/Lives_of_the_Twelve_Caesars']
    Appian
    ['/wiki/Appian']
    Arrian
    ['/wiki/Arrian']
    Lucius Ampelius
    ['/wiki/Liber_Memorialis']
    Dio Cassius
    ['/wiki/Dio_Cassius']
    Herodian
    ['/wiki/Herodian']
    Sextus Julius Africanus
    ['/wiki/Sextus_Julius_Africanus']
    Diogenes Laërtius
    ['/wiki/Diogenes_La%C3%ABrtius']
    Chen Shou
    ['/wiki/Chen_Shou']
    Eusebius of Caesarea
    ['/wiki/Eusebius_of_Caesarea']
    Ammianus Marcellinus
    ['/wiki/Ammianus_Marcellinus']
    Fa-Hien
    ['/wiki/Fa-Hien']
    Rufinus of Aquileia
    ['/wiki/Rufinus_of_Aquileia']
    Philostorgius
    ['/wiki/Philostorgius']
    Socrates of Constantinople
    ['/wiki/Socrates_of_Constantinople']
    Theodoret
    ['/wiki/Theodoret']
    Fan Ye (historian)
    ['/wiki/Fan_Ye_(historian)']
    Priscus
    ['/wiki/Priscus']
    Sozomen
    ['/wiki/Sozomen']
    Salvian
    ['/wiki/Salvian']
    Movses Khorenatsi
    ['/wiki/Movses_Khorenatsi']
    Shen Yue
    ['/wiki/Shen_Yue']
    John Malalas
    ['/wiki/John_Malalas']
    Zosimus
    ['/wiki/Zosimus']
    Procopius
    ['/wiki/Procopius']
    Jordanes
    ['/wiki/Jordanes']
    Gregory of Tours
    ['/wiki/Gregory_of_Tours']
    Wei Zheng
    ['/wiki/Wei_Zheng']
    Baudovinia
    ['/wiki/Baudovinia']
    Yao Silian
    ['/wiki/Yao_Silian']
    Fang Xuanling
    ['/wiki/Fang_Xuanling']
    Adamnan
    ['/wiki/Adamnan']
    Bede
    ['/wiki/Bede']
    Tírechán
    ['/wiki/T%C3%ADrech%C3%A1n']
    Cogitosus
    ['/wiki/Cogitosus']
    Muirchu moccu Machtheni
    ['/wiki/Muirchu_moccu_Machtheni']
    Paul the Deacon
    ['/wiki/Paul_the_Deacon']
    Liu Zhiji
    ['/wiki/Liu_Zhiji']
    Ō no Yasumaro
    ['/wiki/%C5%8C_no_Yasumaro']
    Constantine of Preslav
    ['/wiki/Constantine_of_Preslav']
    Nennius
    ['/wiki/Nennius']
    Martianus Hiberniensis
    ['/wiki/Martianus_Hiberniensis']
    Einhard
    ['/wiki/Einhard']
    Notker of St Gall
    ['/wiki/Notker_of_St_Gall']
    Regino of Prüm
    ['/wiki/Regino_of_Pr%C3%BCm']
    Asser
    ['/wiki/Asser']
    Muhammad al-Tabari
    ['/wiki/Muhammad_ibn_Jarir_al-Tabari']
    Liu Xu
    ['/wiki/Liu_Xu']
    Liutprand of Cremona
    ['/wiki/Liutprand_of_Cremona']
    Li Fang
    ['/wiki/Li_Fang_(Song_dynasty)']
    Heriger of Lobbes
    ['/wiki/Heriger_of_Lobbes']
    Al-Biruni
    ['/wiki/Al-Biruni']
    Thietmar of Merseburg
    ['/wiki/Thietmar_of_Merseburg']
    Ibn Rustah
    ['/wiki/Ibn_Rustah']
    Song Qi
    ['/wiki/Song_Qi']
    Ouyang Xiu
    ['/wiki/Ouyang_Xiu']
    Albert of Aix
    ['/wiki/Albert_of_Aix']
    Nestor the Chronicler
    ['/wiki/Nestor_the_Chronicler']
    Gallus Anonymus
    ['/wiki/Gallus_Anonymus']
    Michael Attaleiates
    ['/wiki/Michael_Attaleiates']
    Michael Psellus
    ['/wiki/Michael_Psellus']
    Sima Guang
    ['/wiki/Sima_Guang']
    Marianus Scotus
    ['/wiki/Marianus_Scotus']
    Guibert of Nogent
    ['/wiki/Guibert_of_Nogent']
    Adam of Bremen
    ['/wiki/Adam_of_Bremen']
    Galbert of Bruges
    ['/wiki/Galbert_of_Bruges']
    Florence of Worcester
    ['/wiki/Florence_of_Worcester']
    Eadmer
    ['/wiki/Eadmer']
    Kim Bu-sik
    ['/wiki/Kim_Bu-sik']
    William of Malmesbury
    ['/wiki/William_of_Malmesbury']
    Symeon of Durham
    ['/wiki/Symeon_of_Durham']
    Anna Comnena
    ['/wiki/Anna_Comnena']
    Usamah ibn Munqidh
    ['/wiki/Usamah_ibn_Munqidh']
    Geoffrey of Monmouth
    ['/wiki/Geoffrey_of_Monmouth']
    Helmold of Bosau
    ['/wiki/Helmold_of_Bosau']
    William of Tyre
    ['/wiki/William_of_Tyre']
    Alured of Beverley
    ['/wiki/Alured_of_Beverley']
    William of Newburgh
    ['/wiki/William_of_Newburgh']
    Svend Aagesen
    ['/wiki/Svend_Aagesen']
    Mohammed al-Baydhaq
    ['/wiki/Mohammed_al-Baydhaq']
    John of Worcester
    ['/wiki/John_of_Worcester']
    Giraldus Cambrensis
    ['/wiki/Giraldus_Cambrensis']
    Wincenty Kadlubek
    ['/wiki/Wincenty_Kadlubek']
    Ambroise
    ['/wiki/Ambroise']
    Geoffroi de Villehardouin
    ['/wiki/Geoffroi_de_Villehardouin']
    Kalhana
    ['/wiki/Kalhana']
    Saxo Grammaticus
    ['/wiki/Saxo_Grammaticus']
    Joannes Zonaras
    ['/wiki/Joannes_Zonaras']
    Nicetas Choniates
    ['/wiki/Nicetas_Choniates']
    Snorri Sturluson
    ['/wiki/Snorri_Sturluson']
    Abdelwahid al-Marrakushi
    ['/wiki/Abdelwahid_al-Marrakushi']
    Ata al-Mulk Juvayni
    ['/wiki/Ata_al-Mulk_Juvayni']
    Ibn al-Khabbaza
    ['/wiki/Ibn_al-Khabbaza']
    Matthew Paris
    ['/wiki/Matthew_Paris']
    Domentijan
    ['/wiki/Domentijan']
    Il-yeon
    ['/wiki/Il-yeon']
    Salimbene di Adam
    ['/wiki/Salimbene_di_Adam']
    Abdelaziz al-Malzuzi
    ['/wiki/Abdelaziz_al-Malzuzi']
    Templar of Tyre
    ['/wiki/Templar_of_Tyre']
    Lê Văn Hưu
    ['/wiki/L%C3%AA_V%C4%83n_H%C6%B0u']
    Adam of Eynsham
    ['/wiki/Adam_of_Eynsham']
    Jean de Joinville
    ['/wiki/Jean_de_Joinville']
    Piers Langtoft
    ['/wiki/Piers_Langtoft']
    Rashid-al-Din Hamadani
    ['/wiki/Rashid-al-Din_Hamadani']
    Giovanni Villani
    ['/wiki/Giovanni_Villani']
    Ibn Idhari
    ['/wiki/Ibn_Idhari']
    Ibn Abi Zar
    ['/wiki/Ibn_Abi_Zar']
    Abdullah Wassaf
    ['/wiki/Wassaf']
    Song Lian
    ['/wiki/Song_Lian']
    Toqto'a
    ['/wiki/Toqto%27a_(Yuan_Dynasty)']
    ibn Khaldun
    ['/wiki/Ibn_Khaldun']
    John Clyn
    ['/wiki/John_Clyn']
    Baldassarre Bonaiuti
    ['/wiki/Baldassarre_Bonaiuti']
    Jean Froissart
    ['/wiki/Jean_Froissart']
    Dietrich of Nieheim
    ['/wiki/Dietrich_of_Nieheim']
    John of Fordun
    ['/wiki/John_of_Fordun']
    Ruaidhri Ó Cianáin
    ['/wiki/Ruaidhri_%C3%93_Cian%C3%A1in']
    Christine de Pizan
    ['/wiki/Christine_de_Pizan']
    Álvar García de Santa María
    ['/wiki/%C3%81lvar_Garc%C3%ADa_de_Santa_Mar%C3%ADa']
    Seán Mór Ó Dubhagáin
    ['/wiki/Se%C3%A1n_M%C3%B3r_%C3%93_Dubhag%C3%A1in']
    Adhamh Ó Cianáin
    ['/wiki/Adhamh_%C3%93_Cian%C3%A1in']
    Ismail ibn al-Ahmar
    ['/wiki/Ismail_ibn_al-Ahmar']
    Giolla Íosa Mór Mac Fhirbhisigh
    ['/wiki/Giolla_%C3%8Dosa_M%C3%B3r_Mac_Fhirbhisigh']
    Zhu Quan
    ['/wiki/Zhu_Quan']
    John Capgrave
    ['/wiki/John_Capgrave']
    Alphonsus A Sancta Maria
    ['/wiki/Alphonsus_A_Sancta_Maria']
    Jan Długosz
    ['/wiki/Jan_D%C5%82ugosz']
    Sharaf ad-Din Ali Yazdi
    ['/wiki/Sharaf_ad-Din_Ali_Yazdi']
    Cathal Óg Mac Maghnusa
    ['/wiki/Cathal_%C3%93g_Mac_Maghnusa']
    Philippe de Commines
    ['/wiki/Philippe_de_Commines']
    Robert Fabyan
    ['/wiki/Robert_Fabyan']
    Albert Krantz
    ['/wiki/Albert_Krantz']
    Hector Boece
    ['/wiki/Hector_Boece']
    Polydore Vergil
    ['/wiki/Polydore_Vergil']
    Sigismund von Herberstein
    ['/wiki/Sigismund_von_Herberstein']
    João de Barros
    ['/wiki/Jo%C3%A3o_de_Barros']
    Niccolò Machiavelli
    ['/wiki/Niccol%C3%B2_Machiavelli']
    Francesco Guicciardini
    ['/wiki/Francesco_Guicciardini']
    Josias Simmler
    ['/wiki/Josias_Simmler']
    Paolo Paruta
    ['/wiki/Paolo_Paruta']
    Raphael Holinshed
    ['/wiki/Raphael_Holinshed']
    Caesar Baronius
    ['/wiki/Caesar_Baronius']
    Abd al-Qadir Bada'uni
    ['/wiki/Abd_al-Qadir_Bada%27uni']
    Abd al-Aziz al-Fishtali
    ['/wiki/Abd_al-Aziz_al-Fishtali']
    Ahmad Ibn al-Qadi
    ['/wiki/Ahmad_Ibn_al-Qadi']
    John Hayward
    ['/wiki/John_Hayward_(historian)']
    Pilip Ballach Ó Duibhgeannáin
    ['/wiki/Pilip_Ballach_%C3%93_Duibhgeann%C3%A1in']
    James Ussher
    ['/wiki/James_Ussher']
    William Bradford
    ['/wiki/William_Bradford_(Plymouth_governor)']
    Bahrey
    ['/wiki/Bahrey']
    Fray Íñigo Abbad y Lasierra
    ['/wiki/Fray_%C3%8D%C3%B1igo_Abbad_y_Lasierra']
    Mohammed Akensus
    ['/wiki/Mohammed_Akensus']
    Archibald Alison
    ['/wiki/Sir_Archibald_Alison,_1st_Baronet']
    Abbasgulu Bakikhanov
    ['/wiki/Abbasgulu_Bakikhanov']
    Teimuraz Bagrationi
    ['/wiki/Teimuraz_Bagrationi']
    Archibald Bower
    ['/wiki/Archibald_Bower']
    Đorđe Branković
    ['/wiki/%C4%90or%C4%91e_Brankovi%C4%87_(count)']
    Mary Bonaventure Browne
    ['/wiki/Mary_Bonaventure_Browne']
    Josiah Burchett
    ['/wiki/Josiah_Burchett']
    Ron Chernow
    ['/wiki/Ron_Chernow']
    Chang Hsüeh-ch'eng
    ['/wiki/Chang_Hs%C3%BCeh-ch%27eng']
    Thomas Carlyle
    ['/wiki/Thomas_Carlyle']
    Chaudhry Afzal Haq
    ['/wiki/Chaudhry_Afzal_Haq']
    Agha Shorish Kashmiri
    ['/wiki/Agha_Shorish_Kashmiri']
    Janbaz Mirza
    ['/wiki/Janbaz_Mirza']
    Simonas Daukantas
    ['/wiki/Simonas_Daukantas']
    Charles Dezobry
    ['/wiki/Charles_Dezobry']
    Mohammed al-Duayf
    ['/wiki/Mohammed_al-Duayf']
    John Colin Dunlop
    ['/wiki/John_Colin_Dunlop']
    Laurence Echard
    ['/wiki/Laurence_Echard']
    Abd al-Rahman al-Fasi
    ['/wiki/Abd_al-Rahman_al-Fasi']
    George Finlay
    ['/wiki/George_Finlay']
    Abd al-Aziz al-Fishtali
    ['/wiki/Abd_al-Aziz_al-Fishtali']
    Francisco Jose Freire
    ['/wiki/Francisco_Jose_Freire']
    Francesco Maria Appendini
    ['/wiki/Francesco_Maria_Appendini']
    Charles du Fresne, sieur du Cange
    ['/wiki/Charles_du_Fresne,_sieur_du_Cange']
    Charlotta Frölich
    ['/wiki/Charlotta_Fr%C3%B6lich']
    Garcilaso de la Vega
    ['/wiki/Inca_Garcilaso_de_la_Vega']
    Erik Gustaf Geijer
    ['/wiki/Erik_Gustaf_Geijer']
    Edward Gibbon
    ['/wiki/Edward_Gibbon']
    George Grote
    ['/wiki/George_Grote']
    Giambattista Vico
    ['/wiki/Giambattista_Vico']
    François Guizot
    ['/wiki/Fran%C3%A7ois_Guizot']
    Edward Hasted
    ['/wiki/Edward_Hasted']
    Sulayman al-Hawwat
    ['/wiki/Sulayman_al-Hawwat']
    Georg Wilhelm Friedrich Hegel
    ['/wiki/Georg_Wilhelm_Friedrich_Hegel']
    Alexander Hewat (or Hewatt)
    ['/wiki/Alexander_Hewat']
    Pieter Corneliszoon Hooft
    ['/wiki/Pieter_Corneliszoon_Hooft']
    Arild Huitfeldt
    ['/wiki/Arild_Huitfeldt']
    David Hume
    ['/wiki/David_Hume']
    Thomas Hutchinson
    ['/wiki/Thomas_Hutchinson_(governor)']
    Mohammed al-Ifrani
    ['/wiki/Mohammed_al-Ifrani']
    Nikolai Mikhailovich Karamzin
    ['/wiki/Nikolai_Mikhailovich_Karamzin']
    Geoffrey Keating
    ['/wiki/Geoffrey_Keating']
    Joachim Lelewel
    ['/wiki/Joachim_Lelewel']
    John Lingard
    ['/wiki/John_Lingard']
    Anton Tomaz Linhart
    ['/wiki/Anton_Tomaz_Linhart']
    F.S.L. Lyons
    ['/wiki/F.S.L._Lyons']
    Dubhaltach MacFhirbhisigh
    ['/wiki/Dubhaltach_MacFhirbhisigh']
    Jules Michelet
    ['/wiki/Jules_Michelet']
    François Mignet
    ['/wiki/Fran%C3%A7ois_Mignet']
    Christian Molbech
    ['/wiki/Christian_Molbech']
    Johann Lorenz Von Mosheim
    ['/wiki/Johann_Lorenz_Von_Mosheim']
    Johannes von Müller
    ['/wiki/Johannes_von_M%C3%BCller']
    Ludovico Antonio Muratori
    ['/wiki/Ludovico_Antonio_Muratori']
    Louis-Sébastien Le Nain de Tillemont
    ['/wiki/Louis-S%C3%A9bastien_Le_Nain_de_Tillemont']
    Barthold Georg Niebuhr
    ['/wiki/Barthold_Georg_Niebuhr']
    Tadhg Óg Ó Cianáin
    ['/wiki/Tadhg_%C3%93g_%C3%93_Cian%C3%A1in']
    Mícheál Ó Cléirigh
    ['/wiki/M%C3%ADche%C3%A1l_%C3%93_Cl%C3%A9irigh']
    Peregrine Ó Duibhgeannain
    ['/wiki/Peregrine_%C3%93_Duibhgeannain']
    Cú Choigcríche Ó Cléirigh
    ['/wiki/C%C3%BA_Choigcr%C3%ADche_%C3%93_Cl%C3%A9irigh']
    Ruaidhrí Ó Flaithbheartaigh
    ['/wiki/Ruaidhr%C3%AD_%C3%93_Flaithbheartaigh']
    Zaharije Orfelin
    ['/wiki/Zaharije_Orfelin']
    Olaus Magnus
    ['/wiki/Olaus_Magnus']
    František Palacký
    ['/wiki/Franti%C5%A1ek_Palack%C3%BD']
    William H. Prescott
    ['/wiki/William_H._Prescott']
    Placido Puccinelli
    ['/wiki/Placido_Puccinelli']
    Mohammed al-Qadiri
    ['/wiki/Mohammed_al-Qadiri']
    Qian Daxin
    ['/wiki/Qian_Daxin']
    Qian Qianyi
    ['/wiki/Qian_Qianyi']
    David Ramsay
    ['/wiki/David_Ramsay_(congressman)']
    Leopold von Ranke
    ['/wiki/Leopold_von_Ranke']
    John F. Richards
    ['/wiki/John_F._Richards']
    Mikhail Shcherbatov
    ['/wiki/Mikhail_Shcherbatov']
    John Strype
    ['/wiki/John_Strype']
    Vasily Tatishchev
    ['/wiki/Vasily_Tatishchev']
    Adolphe Thiers
    ['/wiki/Adolphe_Thiers']
    George Tucker
    ['/wiki/George_Tucker_(politician)']
    Voltaire
    ['/wiki/Voltaire']
    Sir James Ware
    ['/wiki/Sir_James_Ware']
    Yu Deuk-gong
    ['/wiki/Yu_Deuk-gong']
    Abu al-Qasim al-Zayyani
    ['/wiki/Abu_al-Qasim_al-Zayyani']
    Zhang Tingyu
    ['/wiki/Zhang_Tingyu']
    Lord Acton
    ['/wiki/John_Dalberg-Acton,_1st_Baron_Acton']
    Henry Adams
    ['/wiki/Henry_Brooks_Adams']
    Grace Aguilar
    ['/wiki/Grace_Aguilar']
    Charles McLean Andrews
    ['/wiki/Charles_McLean_Andrews']
    Alfred von Arneth
    ['/wiki/Alfred_von_Arneth']
    Mikhail Artamonov
    ['/wiki/Mikhail_Artamonov']
    William Ashley
    ['/wiki/William_Ashley_(economic_historian)']
    Octave Aubry
    ['/wiki/Octave_Aubry']
    François Victor Alphonse Aulard
    ['/wiki/Fran%C3%A7ois_Victor_Alphonse_Aulard']
    Zurab Avalishvili
    ['/wiki/Zurab_Avalishvili']
    Jacques Bainville
    ['/wiki/Jacques_Bainville']
    R. Mildred Barker
    ['/wiki/R._Mildred_Barker']
    Harry Elmer Barnes
    ['/wiki/Harry_Elmer_Barnes']
    Charles Bean
    ['/wiki/Charles_Bean']
    Charles A. Beard
    ['/wiki/Charles_A._Beard']
    Mary Ritter Beard
    ['/wiki/Mary_Ritter_Beard']
    George Bancroft
    ['/wiki/George_Bancroft']
    Wilhelm Barthold
    ['/wiki/Vasily_Bartold']
    Winthrop Pickard Bell
    ['/wiki/Winthrop_Pickard_Bell']
    Hilaire Belloc
    ['/wiki/Hilaire_Belloc']
    Marc Bloch
    ['/wiki/Marc_Bloch']
    Herbert Eugene Bolton
    ['/wiki/Herbert_Eugene_Bolton']
    George Williams Brown
    ['/wiki/George_Williams_Brown']
    Erich Brandenburg
    ['/wiki/Erich_Brandenburg']
    Otto Brunner
    ['/wiki/Otto_Brunner']
    Geoffrey Bruun
    ['/wiki/Geoffrey_Bruun']
    Arthur Bryant
    ['/wiki/Arthur_Bryant']
    Henry Thomas Buckle
    ['/wiki/Henry_Thomas_Buckle']
    Jacob Burckhardt
    ['/wiki/Jacob_Burckhardt']
    John Hill Burton
    ['/wiki/John_Hill_Burton']
    J.B. Bury
    ['/wiki/J.B._Bury']
    Helen Cam
    ['/wiki/Helen_Cam']
    Pierre Caron
    ['/wiki/Pierre_Caron_(historian)']
    E.H. Carr
    ['/wiki/E.H._Carr']
    Antonio Cánovas del Castillo
    ['/wiki/Antonio_C%C3%A1novas_del_Castillo']
    Henri Raymond Casgrain
    ['/wiki/Henri_Raymond_Casgrain']
    Américo Castro
    ['/wiki/Am%C3%A9rico_Castro']
    Bruce Catton
    ['/wiki/Bruce_Catton']
    Cesar de Bazancourt
    ['/wiki/Baron_de_C%C3%A9sar_Bazancourt']
    Nirad C. Chaudhuri
    ['/wiki/Nirad_C._Chaudhuri']
    Boris Chicherin
    ['/wiki/Boris_Chicherin']
    Hiram M. Chittenden
    ['/wiki/Hiram_M._Chittenden']
    Winston Churchill
    ['/wiki/Winston_Churchill']
    Augustin Cochin
    ['/wiki/Augustin_Cochin_(historian)']
    R. G. Collingwood
    ['/wiki/R._G._Collingwood']
    Julian Corbett
    ['/wiki/Julian_Corbett']
    Vladimir Ćorović
    ['/wiki/Vladimir_%C4%86orovi%C4%87']
    Avery Craven
    ['/wiki/Avery_Craven']
    Edward Shepherd Creasy
    ['/wiki/Edward_Shepherd_Creasy']
    Margaret Campbell Speke Cruwys
    ['/wiki/Margaret_Campbell_Speke_Cruwys']
    Felix Dahn
    ['/wiki/Felix_Dahn']
    Angie Debo
    ['/wiki/Angie_Debo']
    Léopold Delisle
    ['/wiki/L%C3%A9opold_Victor_Delisle']
    Bernard DeVoto
    ['/wiki/Bernard_DeVoto']
    William Dodd
    ['/wiki/William_Dodd_(ambassador)']
    David C. Douglas
    ['/wiki/David_C._Douglas']
    Johann Gustav Droysen
    ['/wiki/Johann_Gustav_Droysen']
    Ariel Durant
    ['/wiki/Ariel_Durant']
    Will Durant
    ['/wiki/Will_Durant']
    Mary Anne Everett Green
    ['/wiki/Mary_Anne_Everett_Green']
    Ephraim Emerton
    ['/wiki/Ephraim_Emerton']
    Cyril Falls
    ['/wiki/Cyril_Falls']
    Keith Feiling
    ['/wiki/Keith_Feiling']
    Herbert Feis
    ['/wiki/Herbert_Feis']
    Lucien Febvre
    ['/wiki/Lucien_Febvre']
    Charles Harding Firth
    ['/wiki/Charles_Harding_Firth']
    Walter Lynwood Fleming
    ['/wiki/Walter_Lynwood_Fleming']
    Edward Augustus Freeman
    ['/wiki/Edward_Augustus_Freeman']
    James Anthony Froude
    ['/wiki/James_Anthony_Froude']
    J.F.C. Fuller
    ['/wiki/J.F.C._Fuller']
    Frantz Funck-Brentano
    ['/wiki/Frantz_Funck-Brentano']
    John Sydenham Furnivall
    ['/wiki/John_Sydenham_Furnivall']
    Numa Denis Fustel de Coulanges
    ['/wiki/Numa_Denis_Fustel_de_Coulanges']
    François-Louis Ganshof
    ['/wiki/Fran%C3%A7ois-Louis_Ganshof']
    Samuel Rawson Gardiner
    ['/wiki/Samuel_Rawson_Gardiner']
    Pieter Geyl
    ['/wiki/Pieter_Geyl']
    Lawrence Henry Gipson
    ['/wiki/Lawrence_Henry_Gipson']
    Arthur Giry
    ['/wiki/Arthur_Giry']
    Gustave Glotz
    ['/wiki/Gustave_Glotz']
    George Peabody Gooch
    ['/wiki/George_Peabody_Gooch']
    Timofey Granovsky
    ['/wiki/Timofey_Granovsky']
    John Richard Green
    ['/wiki/John_Richard_Green']
    Lionel Groulx
    ['/wiki/Lionel_Groulx']
    René Grousset
    ['/wiki/Ren%C3%A9_Grousset']
    Louis Halphen
    ['/wiki/Louis_Halphen']
    Clarence H. Haring
    ['/wiki/Clarence_H._Haring']
    Charles H. Haskins
    ['/wiki/Charles_H._Haskins']
    Henri Hauser
    ['/wiki/Henri_Hauser']
    Julien Havet
    ['/wiki/Julien_Havet']
    Paul Hazard
    ['/wiki/Paul_Hazard']
    Eli Heckscher
    ['/wiki/Eli_Heckscher']
    Auguste Himly
    ['/wiki/Auguste_Himly']
    Mihály Horváth
    ['/wiki/Mih%C3%A1ly_Horv%C3%A1th']
    Johan Huizinga
    ['/wiki/Johan_Huizinga']
    Ibn Zaydan
    ['/wiki/Ibn_Zaydan']
    Dmitry Ilovaisky
    ['/wiki/Dmitry_Ilovaisky']
    Harold Innis
    ['/wiki/Harold_Innis']
    Mohammed ibn Jaafar al-Kattani
    ['/wiki/Mohammed_ibn_Jaafar_al-Kattani']
    Muhammad Jaber
    ['/wiki/Muhammad_Jaber']
    William James (naval historian)
    ['/wiki/William_James_(naval_historian)']
    Ivane Javakhishvili
    ['/wiki/Ivane_Javakhishvili']
    Samuel Kamakau
    ['/wiki/Samuel_Kamakau']
    Konstantin Kavelin
    ['/wiki/Konstantin_Kavelin']
    Hans Kelsen
    ['/wiki/Hans_Kelsen']
    Philip Moore Callow Kermode
    ['/wiki/P._M._C._Kermode']
    Alexander William Kinglake
    ['/wiki/Alexander_William_Kinglake']
    William Kingsford
    ['/wiki/William_Kingsford']
    Vasily Klyuchevsky
    ['/wiki/Vasily_Klyuchevsky']
    David Knowles
    ['/wiki/David_Knowles_(scholar)']
    Dudley Wright Knox
    ['/wiki/Dudley_Wright_Knox']
    Ludwig von Köchel
    ['/wiki/Ludwig_von_K%C3%B6chel']
    Mihail Kogălniceanu
    ['/wiki/Mihail_Kog%C4%83lniceanu']
    Hans Kohn
    ['/wiki/Hans_Kohn']
    Nikodim Kondakov
    ['/wiki/Nikodim_Kondakov']
    Nikolay Kostomarov
    ['/wiki/Nikolay_Kostomarov']
    Godefroid Kurth
    ['/wiki/Godefroid_Kurth']
    Leonard Woods Labaree
    ['/wiki/Leonard_Woods_Labaree']
    William L. Langer
    ['/wiki/William_L._Langer']
    John Knox Laughton
    ['/wiki/John_Knox_Laughton']
    Ernest Lavisse
    ['/wiki/Ernest_Lavisse']
    Georges Lefebvre
    ['/wiki/Georges_Lefebvre']
    Liang Qichao
    ['/wiki/Liang_Qichao']
    B.H. Liddell Hart
    ['/wiki/B.H._Liddell_Hart']
    John Edward Lloyd
    ['/wiki/John_Edward_Lloyd']
    Ferdinand Lot
    ['/wiki/Ferdinand_Lot']
    Arthur R.M. Lower
    ['/wiki/Arthur_R.M._Lower']
    Thomas Macaulay
    ['/wiki/Thomas_Macaulay']
    William Archibald Mackintosh
    ['/wiki/William_Archibald_Mackintosh']
    J. D. Mackie
    ['/wiki/J._D._Mackie']
    Frederic William Maitland
    ['/wiki/Frederic_William_Maitland']
    Alfred Thayer Mahan
    ['/wiki/Alfred_Thayer_Mahan']
    Ramesh Chandra Majumdar
    ['/wiki/Ramesh_Chandra_Majumdar']
    J.A.R. Marriott
    ['/wiki/John_Marriott_(British_politician)']
    Albert Mathiez
    ['/wiki/Albert_Mathiez']
    Karl Marx
    ['/wiki/Karl_Marx']
    Friedrich Meinecke
    ['/wiki/Friedrich_Meinecke']
    Krste Misirkov
    ['/wiki/Krste_Misirkov']
    Auguste Molinier
    ['/wiki/Auguste_Molinier']
    Theodor Mommsen
    ['/wiki/Theodor_Mommsen']
    Indro Montanelli
    ['/wiki/Indro_Montanelli']
    Alfred Morel-Fatio
    ['/wiki/Alfred_Morel-Fatio']
    Samuel Eliot Morison
    ['/wiki/Samuel_Eliot_Morison']
    Lewis Mumford
    ['/wiki/Lewis_Mumford']
    Lewis Bernstein Namier
    ['/wiki/Lewis_Bernstein_Namier']
    Ahmad ibn Khalid al-Nasiri
    ['/wiki/Ahmad_ibn_Khalid_al-Nasiri']
    J. E. Neale
    ['/wiki/J._E._Neale']
    Allan Nevins
    ['/wiki/Allan_Nevins']
    A. P. Newton
    ['/wiki/A._P._Newton']
    Stojan Novaković
    ['/wiki/Stojan_Novakovi%C4%87']
    Charles Oman
    ['/wiki/Charles_Oman']
    Herbert L. Osgood
    ['/wiki/Herbert_L._Osgood']
    Cesare Paoli
    ['/wiki/Cesare_Paoli']
    Gaston Paris
    ['/wiki/Gaston_Paris']
    Herbert Paul
    ['/wiki/Herbert_Paul']
    Henry Francis Pelham
    ['/wiki/Henry_Francis_Pelham']
    Samuel W. Pennypacker
    ['/wiki/Samuel_W._Pennypacker']
    Dexter Perkins
    ['/wiki/Dexter_Perkins']
    Henri Pirenne
    ['/wiki/Henri_Pirenne']
    Sergey Platonov
    ['/wiki/Sergey_Platonov']
    Ivy Pinchbeck
    ['/wiki/Ivy_Pinchbeck']
    Eileen Power
    ['/wiki/Eileen_Power']
    F. M. Powicke
    ['/wiki/F._M._Powicke']
    H. F. M. Prescott
    ['/wiki/H._F._M._Prescott']
    Datto Vaman Potdar
    ['/wiki/Datto_Vaman_Potdar']
    Jules Quicherat
    ['/wiki/Jules_Etienne_Joseph_Quicherat']
    William Pember Reeves
    ['/wiki/William_Pember_Reeves']
    Pierre Renouvin
    ['/wiki/Pierre_Renouvin']
    James Riker
    ['/wiki/James_Riker']
    B. H. Roberts
    ['/wiki/B._H._Roberts']
    James Harvey Robinson
    ['/wiki/James_Harvey_Robinson']
    Theodore Roosevelt
    ['/wiki/Theodore_Roosevelt']
    Simon Rutar
    ['/wiki/Simon_Rutar']
    Ilarion Ruvarac
    ['/wiki/Ilarion_Ruvarac']
    Abram L. Sachar
    ['/wiki/Abram_L._Sachar']
    George Sarton
    ['/wiki/George_Sarton']
    Gustave Schlumberger
    ['/wiki/Gustave_Schlumberger']
    John Robert Seeley
    ['/wiki/John_Robert_Seeley']
    Sergey Solovyov
    ['/wiki/Sergey_Solovyov']
    Govind Sakharam Sardesai
    ['/wiki/Govind_Sakharam_Sardesai']
    Adam Shortt
    ['/wiki/Adam_Shortt']
    Goldwin Smith
    ['/wiki/Goldwin_Smith']
    Oswald Spengler
    ['/wiki/Oswald_Spengler']
    Shin Chaeho
    ['/wiki/Shin_Chaeho']
    Frank Stenton
    ['/wiki/Frank_Stenton']
    Doris Mary Stenton
    ['/wiki/Doris_Mary_Stenton']
    William Stubbs
    ['/wiki/William_Stubbs']
    Hippolyte Taine
    ['/wiki/Hippolyte_Taine']
    Frank Bigelow Tarbell
    ['/wiki/Frank_Bigelow_Tarbell']
    Yevgeny Tarle
    ['/wiki/Yevgeny_Tarle']
    A. Wyatt Tilby
    ['/wiki/A._Wyatt_Tilby']
    Alexis de Tocqueville
    ['/wiki/Alexis_de_Tocqueville']
    Zacharias Topelius
    ['/wiki/Zacharias_Topelius']
    Thomas Frederick Tout
    ['/wiki/Thomas_Frederick_Tout']
    Arnold J. Toynbee
    ['/wiki/Arnold_J._Toynbee']
    Heinrich Gotthard von Treitschke
    ['/wiki/Heinrich_Gotthard_von_Treitschke']
    George Macaulay Trevelyan
    ['/wiki/George_Macaulay_Trevelyan']
    Mikheil Tsereteli
    ['/wiki/Mikheil_Tsereteli']
    Frank Underhill
    ['/wiki/Frank_Underhill']
    Paul Vinogradoff
    ['/wiki/Paul_Vinogradoff']
    Spencer Walpole
    ['/wiki/Spencer_Walpole']
    Curt Weibull
    ['/wiki/Curt_Weibull']
    Lauritz Weibull
    ['/wiki/Lauritz_Weibull']
    Charles Webster
    ['/wiki/Charles_Webster_(historian)']
    Mary Wilhelmine Williams
    ['/wiki/Mary_Wilhelmine_Williams']
    Spenser Wilkinson
    ['/wiki/Spenser_Wilkinson']
    James A. Williamson
    ['/wiki/James_Williamson_(historian)']
    Justin Winsor
    ['/wiki/Justin_Winsor']
    Llewellyn Woodward
    ['/wiki/Llewellyn_Woodward']
    George MacKinnon Wrong
    ['/wiki/George_MacKinnon_Wrong']
    Yi Byeongdo
    ['/wiki/Yi_Byeongdo']
    Faddei Zielinski
    ['/wiki/Faddei_Zielinski']
    Raouf Abbas
    ['/wiki/Raouf_Abbas']
    Irving Abella
    ['/wiki/Irving_Abella']
    Aberjhani
    ['/wiki/Aberjhani']
    David Abulafia
    ['/wiki/David_Abulafia']
    Ezequiel Adamovsky
    ['/wiki/Ezequiel_Adamovsky']
    Donald Adamson
    ['/wiki/Donald_Adamson']
    Teodoro Agoncillo
    ['/wiki/Teodoro_Agoncillo']
    Robert G. Albion
    ['/wiki/Robert_G._Albion']
    Dean C. Allard
    ['/wiki/Dean_C._Allard']
    Robert C. Allen
    ['/wiki/Robert_C._Allen']
    Gar Alperovitz
    ['/wiki/Gar_Alperovitz']
    Ida Altman
    ['/wiki/Ida_Altman']
    Abbas Amanat
    ['/wiki/Abbas_Amanat']
    Mor Altshuler
    ['/wiki/Mor_Altshuler']
    Henri Amouroux
    ['/wiki/Henri_Amouroux']
    Stephen Ambrose
    ['/wiki/Stephen_Ambrose']
    Perry Anderson
    ['/wiki/Perry_Anderson']
    Joyce Appleby
    ['/wiki/Joyce_Appleby']
    Herbert Aptheker
    ['/wiki/Herbert_Aptheker']
    Leonie Archer
    ['/wiki/Leonie_Archer']
    Philippe Ariès
    ['/wiki/Philippe_Ari%C3%A8s']
    Karen Armstrong
    ['/wiki/Karen_Armstrong']
    Leonard J. Arrington
    ['/wiki/Leonard_J._Arrington']
    Thomas Asbridge
    ['/wiki/Thomas_Asbridge']
    Maurice Ashley
    ['/wiki/Maurice_Ashley_(historian)']
    Paul Avrich
    ['/wiki/Paul_Avrich']
    Ali Azaykou
    ['/wiki/Ali_Azaykou']
    Eiichiro Azuma
    ['/wiki/Eiichiro_Azuma']
    Nigel Bagnall
    ['/wiki/Nigel_Bagnall']
    Bernard Bailyn
    ['/wiki/Bernard_Bailyn']
    David E. Barclay
    ['/wiki/David_E._Barclay']
    Juliet Barker
    ['/wiki/Juliet_Barker']
    Frank Barlow
    ['/wiki/Frank_Barlow_(historian)']
    Linda Diane Barnes
    ['/wiki/Linda_Diane_Barnes']
    G.W.S. Barrow
    ['/wiki/G.W.S._Barrow']
    H. Arnold Barton
    ['/wiki/H._Arnold_Barton']
    Paul R. Bartrop
    ['/wiki/Paul_R._Bartrop']
    Jacques Barzun
    ['/wiki/Jacques_Barzun']
    Jorge Basadre
    ['/wiki/Jorge_Basadre']
    Hanna Batatu
    ['/wiki/Hanna_Batatu']
    K. Jack Bauer
    ['/wiki/K._Jack_Bauer']
    Yehuda Bauer
    ['/wiki/Yehuda_Bauer']
    Stephen B. Baxter
    ['/wiki/Stephen_B._Baxter']
    David Bebbington
    ['/wiki/David_Bebbington']
    Antony Beevor
    ['/wiki/Antony_Beevor']
    James Belich
    ['/wiki/James_Belich_(historian)']
    Abdelmajid Benjelloun
    ['/wiki/Abdelmajid_Benjelloun_(historian)']
    Laurence Bergreen
    ['/wiki/Laurence_Bergreen']
    Isaiah Berlin
    ['/wiki/Isaiah_Berlin']
    Michael Beschloss
    ['/wiki/Michael_Beschloss']
    Nicholas Bethell
    ['/wiki/Nicholas_Bethell']
    Anthony Birley
    ['/wiki/Anthony_Birley']
    David Blackbourn
    ['/wiki/David_Blackbourn']
    Geoffrey Blainey
    ['/wiki/Geoffrey_Blainey']
    Gisela Bock
    ['/wiki/Gisela_Bock']
    Brian Bond
    ['/wiki/Brian_Bond']
    Daniel J. Boorstin
    ['/wiki/Daniel_J._Boorstin']
    Georges Bordonove
    ['/wiki/Georges_Bordonove']
    John Boswell
    ['/wiki/John_Boswell']
    Robert Bothwell
    ['/wiki/Robert_Bothwell']
    Gérard Bouchard
    ['/wiki/G%C3%A9rard_Bouchard']
    Joanna Bourke
    ['/wiki/Joanna_Bourke']
    Paul S. Boyer
    ['/wiki/Paul_S._Boyer']
    Karl Dietrich Bracher
    ['/wiki/Karl_Dietrich_Bracher']
    Jim Bradbury
    ['/wiki/Jim_Bradbury']
    James C. Bradford
    ['/wiki/James_C._Bradford']
    David Brading
    ['/wiki/David_Brading']
    William Brandon
    ['/wiki/William_Brandon_(author)']
    Fernand Braudel
    ['/wiki/Fernand_Braudel']
    Ahron Bregman
    ['/wiki/Ahron_Bregman']
    Carl Bridenbaugh
    ['/wiki/Carl_Bridenbaugh']
    Timothy Brook
    ['/wiki/Timothy_Brook_(historian)']
    Martin Broszat
    ['/wiki/Martin_Broszat']
    Peter Brown
    ['/wiki/Peter_Brown_(historian)']
    Christopher Browning
    ['/wiki/Christopher_Browning']
    Alan Bullock
    ['/wiki/Alan_Bullock']
    Peter Burke
    ['/wiki/Peter_Burke_(historian)']
    Briton C. Busch
    ['/wiki/Briton_C._Busch']
    Richard Bushman
    ['/wiki/Richard_Bushman']
    Herbert Butterfield
    ['/wiki/Herbert_Butterfield']
    Angus Calder
    ['/wiki/Angus_Calder']
    Julio Caro Baroja
    ['/wiki/Julio_Caro_Baroja']
    Sir Raymond Carr
    ['/wiki/Raymond_Carr']
    Paul Cartledge
    ['/wiki/Paul_Cartledge']
    Lionel Casson
    ['/wiki/Lionel_Casson']
    Boris Celovsky
    ['/wiki/Borivoj_Celovsky']
    Howard I. Chapelle
    ['/wiki/Howard_I._Chapelle']
    Maher Charif
    ['/wiki/Maher_Charif']
    Iris Chang
    ['/wiki/Iris_Chang']
    Louis Chevalier
    ['/wiki/Louis_Chevalier']
    Thomas Childers
    ['/wiki/Thomas_Childers']
    I. R. Christie
    ['/wiki/I._R._Christie']
    Alexander Campbell Cheyne
    ['/wiki/Alexander_Campbell_Cheyne']
    Satyabrata Rai Chowdhuri
    ['/wiki/Satyabrata_Rai_Chowdhuri']
    Alan Clark
    ['/wiki/Alan_Clark']
    Christopher Clark
    ['/wiki/Chris_Clark_(historian)']
    J.C.D. Clark
    ['/wiki/J.C.D._Clark']
    Manning Clark
    ['/wiki/Manning_Clark']
    Patrick Collinson
    ['/wiki/Patrick_Collinson']
    Robert Conquest
    ['/wiki/Robert_Conquest']
    Margaret Conrad
    ['/wiki/Margaret_Conrad']
    Vladimir Ćorović
    ['/wiki/Vladimir_%C4%86orovi%C4%87']
    Peter Cottrell
    ['/wiki/Peter_Cottrell']
    Gordon A. Craig
    ['/wiki/Gordon_A._Craig']
    Donald Creighton
    ['/wiki/Donald_Creighton']
    Vincent Cronin
    ['/wiki/Vincent_Cronin']
    William Cronon
    ['/wiki/William_Cronon']
    Pamela Kyle Crossley
    ['/wiki/Pamela_Kyle_Crossley']
    Dan Cruickshank
    ['/wiki/Dan_Cruickshank']
    Barry Cunliffe
    ['/wiki/Barry_Cunliffe']
    John Shelton Curtiss
    ['/wiki/John_Shelton_Curtiss']
    Robert Dallek
    ['/wiki/Robert_Dallek']
    Vahakn N. Dadrian
    ['/wiki/Vahakn_N._Dadrian']
    David B. Danbom
    ['/wiki/David_B._Danbom']
    Ahmad Hasan Dani
    ['/wiki/Ahmad_Hasan_Dani']
    Robert Darnton
    ['/wiki/Robert_Darnton']
    Lucy Dawidowicz
    ['/wiki/Lucy_Dawidowicz']
    Saul David
    ['/wiki/Saul_David']
    John Davies
    ['/wiki/John_Davies_(historian)']
    Norman Davies
    ['/wiki/Norman_Davies']
    Natalie Zemon Davis
    ['/wiki/Natalie_Zemon_Davis']
    Kenneth S. Davis
    ['/wiki/Kenneth_S._Davis']
    R.H.C. Davis
    ['/wiki/Ralph_Henry_Carless_Davis']
    David Day
    ['/wiki/David_Day_(historian)']
    Renzo De Felice
    ['/wiki/Renzo_De_Felice']
    Len Deighton
    ['/wiki/Len_Deighton']
    Carl N. Degler
    ['/wiki/Carl_N._Degler']
    Esther Delisle
    ['/wiki/Esther_Delisle']
    Jean Delumeau
    ['/wiki/Jean_Delumeau']
    Marcel Detienne
    ['/wiki/Marcel_Detienne']
    Alexandre Deulofeu
    ['/wiki/Alexandre_Deulofeu']
    Isaac Deutscher
    ['/wiki/Isaac_Deutscher']
    Tom M. Devine
    ['/wiki/Tom_M._Devine']
    Wu Di
    ['/wiki/Wu_Di_(film_critic_and_historian)']
    Igor M. Diakonov
    ['/wiki/Igor_M._Diakonov']
    David Herbert Donald
    ['/wiki/David_Herbert_Donald']
    Gordon Donaldson
    ['/wiki/Gordon_Donaldson']
    Susan Doran
    ['/wiki/Susan_Doran']
    William Doyle
    ['/wiki/William_Doyle_(historian)']
    Georges Duby
    ['/wiki/Georges_Duby']
    William S. Dudley
    ['/wiki/William_S._Dudley']
    Robert Dudley Edwards
    ['/wiki/Robert_Dudley_Edwards']
    Eamon Duffy
    ['/wiki/Eamon_Duffy']
    A. Hunter Dupree
    ['/wiki/A._Hunter_Dupree']
    Trevor Dupuy
    ['/wiki/Trevor_Dupuy']
    Jean-Baptiste Duroselle
    ['/wiki/Jean-Baptiste_Duroselle']
    Harold James Dyos
    ['/wiki/Harold_James_Dyos']
    Elizabeth Eisenstein
    ['/wiki/Elizabeth_Eisenstein']
    Geoff Eley
    ['/wiki/Geoff_Eley']
    John Elliott
    ['/wiki/John_Elliott_(historian)']
    Joseph J. Ellis
    ['/wiki/Joseph_J._Ellis']
    Geoffrey Elton
    ['/wiki/Geoffrey_Elton']
    Peter Englund
    ['/wiki/Peter_Englund']
    Robert Malcolm Errington
    ['/wiki/Robert_Malcolm_Errington']
    Richard J. Evans
    ['/wiki/Richard_J._Evans']
    Alf Evers
    ['/wiki/Alf_Evers']
    Brian Farrell
    ['/wiki/Brian_Farrell_(broadcaster)']
    John Lister Illingworth Fennell
    ['/wiki/John_Lister_Illingworth_Fennell']
    Niall Ferguson
    ['/wiki/Niall_Ferguson']
    Božidar Ferjančić
    ['/wiki/Bo%C5%BEidar_Ferjan%C4%8Di%C4%87']
    Marc Ferro
    ['/wiki/Marc_Ferro']
    Joachim Fest
    ['/wiki/Joachim_Fest']
    David Feuerwerker
    ['/wiki/David_Feuerwerker']
    Heinrich Fichtenau
    ['/wiki/Heinrich_Fichtenau']
    David Kenneth Fieldhouse
    ['/wiki/David_Kenneth_Fieldhouse']
    Orlando Figes
    ['/wiki/Orlando_Figes']
    Robert O. Fink
    ['/wiki/Robert_O._Fink']
    Moses Finley
    ['/wiki/Moses_Finley']
    David Hackett Fischer
    ['/wiki/David_Hackett_Fischer']
    Fritz Fischer
    ['/wiki/Fritz_Fischer_(historian)']
    Frances FitzGerald
    ['/wiki/Frances_FitzGerald_(journalist)']
    Judith Flanders
    ['/wiki/Judith_Flanders']
    Robert Fogel
    ['/wiki/Robert_Fogel']
    Eric Foner
    ['/wiki/Eric_Foner']
    Shelby Foote
    ['/wiki/Shelby_Foote']
    Amanda Foreman
    ['/wiki/Amanda_Foreman_(historian)']
    Michel Foucault
    ['/wiki/Michel_Foucault']
    Jo Fox
    ['/wiki/Jo_Fox']
    Robin Lane Fox
    ['/wiki/Robin_Lane_Fox']
    Stephen Fox
    ['/wiki/Stephen_Fox_(author/educator)']
    Elizabeth Fox-Genovese
    ['/wiki/Elizabeth_Fox-Genovese']
    Walter Frank
    ['/wiki/Walter_Frank']
    H. Bruce Franklin
    ['/wiki/H._Bruce_Franklin']
    Antonia Fraser
    ['/wiki/Antonia_Fraser']
    Frank Freidel
    ['/wiki/Frank_Freidel']
    Joseph Friedenson
    ['/wiki/Joseph_Friedenson']
    Henry Friedlander
    ['/wiki/Henry_Friedlander']
    Saul Friedländer
    ['/wiki/Saul_Friedl%C3%A4nder']
    Sheppard Frere
    ['/wiki/Sheppard_Frere']
    David Fromkin
    ['/wiki/David_Fromkin']
    Bruno Fuligni
    ['/wiki/Bruno_Fuligni']
    Francis Fukuyama
    ['/wiki/Francis_Fukuyama']
    François Furet
    ['/wiki/Fran%C3%A7ois_Furet']
    Femme Gaastra
    ['/wiki/Femme_Gaastra']
    John Lewis Gaddis
    ['/wiki/John_Lewis_Gaddis']
    Lloyd Gardner
    ['/wiki/Lloyd_Gardner']
    Peter Gay
    ['/wiki/Peter_Gay']
    Eugene Genovese
    ['/wiki/Eugene_Genovese']
    François Géré
    ['/wiki/Fran%C3%A7ois_G%C3%A9r%C3%A9']
    Imanuel Geiss
    ['/wiki/Imanuel_Geiss']
    Christian Gerlach
    ['/wiki/Christian_Gerlach']
    N.H. Gibbs
    ['/wiki/N.H._Gibbs']
    William Gibson
    ['/wiki/William_Gibson_(historian)']
    Martin Gilbert
    ['/wiki/Martin_Gilbert']
    Carlo Ginzburg
    ['/wiki/Carlo_Ginzburg']
    Jan Glete
    ['/wiki/Jan_Glete']
    Eric F. Goldman
    ['/wiki/Eric_F._Goldman']
    James Goldrick
    ['/wiki/James_Goldrick']
    Adrian Goldsworthy
    ['/wiki/Adrian_Goldsworthy']
    Brison D. Gooch
    ['/wiki/Brison_D._Gooch']
    Doris Kearns Goodwin
    ['/wiki/Doris_Kearns_Goodwin']
    Andrew Gordon
    ['/wiki/Andrew_Gordon_(naval_historian)']
    Gerald S. Graham
    ['/wiki/Gerald_S._Graham']
    Jack Granatstein
    ['/wiki/Jack_Granatstein']
    Peter Green
    ['/wiki/Peter_Green_(historian)']
    Vivian H.H. Green
    ['/wiki/Rev._Vivian_Green']
    John Robert Greene
    ['/wiki/John_Robert_Greene']
    Roger D. Griffin
    ['/wiki/Roger_D._Griffin']
    Ranajit Guha
    ['/wiki/Ranajit_Guha']
    Ramchandra Guha
    ['/wiki/Ramchandra_Guha']
    Lev Gumilyov
    ['/wiki/Lev_Gumilyov']
    Oliver Gurney
    ['/wiki/Oliver_Gurney']
    John Guy
    ['/wiki/John_Guy_(historian)']
    Irfan Habib
    ['/wiki/Irfan_Habib']
    Sheldon Hackney
    ['/wiki/Sheldon_Hackney']
    Kenneth J. Hagan
    ['/wiki/Kenneth_J._Hagan']
    Claude Hall
    ['/wiki/Claude_Hall']
    John Whitney Hall
    ['/wiki/John_Whitney_Hall']
    Bruce Barrymore Halpenny
    ['/wiki/Bruce_Barrymore_Halpenny']
    N. G. L. Hammond
    ['/wiki/N._G._L._Hammond']
    Victor Davis Hanson
    ['/wiki/Victor_Davis_Hanson']
    Syed Nomanul Haq
    ['/wiki/Syed_Nomanul_Haq']
    Dick Harrison
    ['/wiki/Dick_Harrison']
    Peter Harrison
    ['/wiki/Peter_Harrison_(historian)']
    Max Hastings
    ['/wiki/Max_Hastings']
    John Hattendorf
    ['/wiki/John_Hattendorf']
    Ragnhild Hatton
    ['/wiki/Ragnhild_Hatton']
    Denys Hay
    ['/wiki/Denys_Hay']
    John Daniel Hayes
    ['/wiki/John_Daniel_Hayes']
    Ingo Heidbrink
    ['/wiki/Ingo_Heidbrink']
    Jeffrey Herf
    ['/wiki/Jeffrey_Herf']
    Arthur Herman
    ['/wiki/Arthur_Herman']
    Michael Hicks
    ['/wiki/Michael_Hicks_(historian)']
    Raul Hilberg
    ['/wiki/Raul_Hilberg']
    Klaus Hildebrand
    ['/wiki/Klaus_Hildebrand']
    Christopher Hill
    ['/wiki/Christopher_Hill_(historian)']
    Andreas Hillgruber
    ['/wiki/Andreas_Hillgruber']
    Richard L. Hills
    ['/wiki/Richard_L._Hills']
    Gertrude Himmelfarb
    ['/wiki/Gertrude_Himmelfarb']
    Harry Hinsley
    ['/wiki/Harry_Hinsley']
    Eric Hobsbawm
    ['/wiki/Eric_Hobsbawm']
    Marshall Hodgson
    ['/wiki/Marshall_Hodgson']
    Richard Hofstadter
    ['/wiki/Richard_Hofstadter']
    Peter Hoffmann
    ['/wiki/Peter_Hoffmann_(historian)']
    David Hoggan
    ['/wiki/David_Hoggan']
    Hajo Holborn
    ['/wiki/Hajo_Holborn']
    Tom Holland
    ['/wiki/Tom_Holland_(author)']
    C. Warren Hollister
    ['/wiki/C._Warren_Hollister']
    George Holmes (professor)
    ['/wiki/George_Holmes_(professor)']
    Richard Holmes
    ['/wiki/Richard_Holmes_(historian)']
    Ed Hooper
    ['/wiki/Ed_Hooper']
    A.G. Hopkins
    ['/wiki/A.G._Hopkins']
    Keith Hopkins
    ['/wiki/Keith_hopkins']
    Albert Hourani
    ['/wiki/Albert_Hourani']
    Youssef Hourany
    ['/wiki/Youssef_Hourany']
    Daniel Horowitz
    ['/wiki/Daniel_Horowitz']
    Helen Lefkowitz Horowitz
    ['/wiki/Helen_Lefkowitz_Horowitz']
    Michiel Horn
    ['/wiki/Michiel_Horn']
    Alistair Horne
    ['/wiki/Alistair_Horne']
    Michael Howard
    ['/wiki/Michael_Howard_(historian)']
    Robert Hughes
    ['/wiki/Robert_Hughes_(critic)']
    Andrew Hunt
    ['/wiki/Andrew_Hunt_(historian)']
    Tristram Hunt
    ['/wiki/Tristram_Hunt']
    Mark C. Hunter
    ['/wiki/Mark_C._Hunter']
    Halil Inalcik
    ['/wiki/Halil_Inalcik']
    Iqbal, Sheikh Mohammad
    ['/wiki/Sheikh_Mohammad_Iqbal']
    Jonathan Israel
    ['/wiki/Jonathan_Israel']
    Eberhard Jäckel
    ['/wiki/Eberhard_J%C3%A4ckel']
    Julian T. Jackson
    ['/wiki/Julian_T._Jackson']
    Harold James
    ['/wiki/Harold_James_(historian)']
    Nikoloz Janashia
    ['/wiki/Nikoloz_Janashia']
    Simon Janashia
    ['/wiki/Simon_Janashia']
    Marius Jansen
    ['/wiki/Marius_Jansen']
    Pawel Jasienica
    ['/wiki/Pawel_Jasienica']
    Merrill Jensen
    ['/wiki/Merrill_Jensen_(historian)']
    Richard J. Jensen
    ['/wiki/Richard_J._Jensen']
    Khasnor Johan
    ['/wiki/Khasnor_Johan']
    Paul Johnson
    ['/wiki/Paul_Johnson_(writer)']
    Robert Erwin Johnson
    ['/wiki/Robert_Erwin_Johnson']
    Mauno Jokipii
    ['/wiki/Mauno_Jokipii']
    A.H.M. Jones
    ['/wiki/Arnold_Hugh_Martin_Jones']
    Gwyn Jones
    ['/wiki/Gwyn_Jones_(author)']
    George Hilton Jones III
    ['/wiki/George_Hilton_Jones,_III_(historian)']
    Loe de Jong
    ['/wiki/Loe_de_Jong']
    Tony Judt
    ['/wiki/Tony_Judt']
    David S. Katz
    ['/wiki/David_S._Katz']
    Donald Kagan
    ['/wiki/Donald_Kagan']
    Elie Kedourie
    ['/wiki/Elie_Kedourie']
    Rod Kedward
    ['/wiki/Rod_Kedward']
    John Keegan
    ['/wiki/John_Keegan']
    Nushiravan Keihanizadeh
    ['/wiki/Nushiravan_Keihanizadeh']
    John H. Kemble
    ['/wiki/John_H._Kemble']
    Paul Murray Kendall
    ['/wiki/Paul_Murray_Kendall']
    Elizabeth Topham Kennan
    ['/wiki/Elizabeth_Topham_Kennan']
    George F. Kennan
    ['/wiki/George_F._Kennan']
    James Kennedy
    ['/wiki/James_Kennedy_(historian)']
    Paul Kennedy
    ['/wiki/Paul_Kennedy']
    W. Hudson Kensel
    ['/wiki/W._Hudson_Kensel']
    Ian Kershaw
    ['/wiki/Ian_Kershaw']
    Daniel J. Kevles
    ['/wiki/Daniel_J._Kevles']
    Khan Roshan Khan
    ['/wiki/Khan_Roshan_Khan']
    Kim Jung-bae
    ['/wiki/Kim_Jung-bae']
    Michael King
    ['/wiki/Michael_King']
    Patrick Kinross
    ['/wiki/Patrick_Kinross']
    Henry Kissinger
    ['/wiki/Henry_Kissinger']
    Martin Kitchen
    ['/wiki/Martin_Kitchen']
    Simon Kitson
    ['/wiki/Simon_Kitson']
    Matti Klinge
    ['/wiki/Matti_Klinge']
    Felix Klos
    ['/wiki/Felix_Klos']
    R.J.B. Knight
    ['/wiki/R.J.B._Knight']
    Yuri Knorozov
    ['/wiki/Yuri_Knorozov']
    Eberhard Kolb
    ['/wiki/Eberhard_Kolb']
    Gabriel Kolko
    ['/wiki/Gabriel_Kolko']
    Claudia Koonz
    ['/wiki/Claudia_Koonz']
    Andrey Korotayev
    ['/wiki/Andrey_Korotayev']
    Ernst Kossmann
    ['/wiki/Ernst_Kossmann']
    Philip A. Kuhn
    ['/wiki/Philip_A._Kuhn']
    Thomas Kuhn
    ['/wiki/Thomas_Kuhn']
    Myoma Myint Kywe
    ['/wiki/Myoma_Myint_Kywe']
    K.S. Lal
    ['/wiki/K.S._Lal']
    Benjamin Woods Labaree
    ['/wiki/Benjamin_Woods_Labaree']
    Brij Lal
    ['/wiki/Brij_Lal_(historian)']
    Abdallah Laroui
    ['/wiki/Abdallah_Laroui']
    Leopold Labedz
    ['/wiki/Leopold_Labedz']
    Andrew Lambert
    ['/wiki/Andrew_Lambert']
    Harold Lamb
    ['/wiki/Harold_Lamb']
    Ricardo Lancaster-Jones y Verea
    ['/wiki/Ricardo_Lancaster-Jones_y_Verea']
    David Lavender
    ['/wiki/David_Lavender']
    Walter LaFeber
    ['/wiki/Walter_LaFeber']
    Daniel Leab
    ['/wiki/Daniel_Leab']
    Jacques Le Goff
    ['/wiki/Jacques_Le_Goff']
    Robert Leckie
    ['/wiki/Robert_Leckie_(author)']
    William Leuchtenburg
    ['/wiki/William_Leuchtenburg']
    Barbara Levick
    ['/wiki/Barbara_Levick']
    David Levering Lewis
    ['/wiki/David_Levering_Lewis']
    Emmanuel Le Roy Ladurie
    ['/wiki/Emmanuel_Le_Roy_Ladurie']
    Lee Ki-baek
    ['/wiki/Lee_Ki-baek']
    Li Ao
    ['/wiki/Li_Ao']
    Leon F. Litwack
    ['/wiki/Leon_F._Litwack']
    Xinru Liu
    ['/wiki/Xinru_Liu']
    Mario Liverani
    ['/wiki/Mario_Liverani']
    Radoš Ljušić
    ['/wiki/Rado%C5%A1_Lju%C5%A1i%C4%87']
    David Loades
    ['/wiki/David_Loades']
    James W. Loewen
    ['/wiki/James_W._Loewen']
    Elizabeth Longford
    ['/wiki/Elizabeth_Pakenham,_Countess_of_Longford']
    Erik Lönnroth
    ['/wiki/Erik_L%C3%B6nnroth']
    Walter Lord
    ['/wiki/Walter_Lord']
    John Lukacs
    ['/wiki/John_Lukacs']
    Charles B. MacDonald
    ['/wiki/Charles_B._MacDonald']
    Stuart Macintyre
    ['/wiki/Stuart_Macintyre']
    Forrest McDonald
    ['/wiki/Forrest_McDonald']
    K. B. McFarlane
    ['/wiki/K._B._McFarlane']
    Ross McKibbin
    ['/wiki/Ross_McKibbin']
    Rosamond McKitterick
    ['/wiki/Rosamond_McKitterick']
    Margaret MacMillan
    ['/wiki/Margaret_MacMillan']
    William Miller Macmillan
    ['/wiki/William_Miller_Macmillan']
    Ramsay MacMullen
    ['/wiki/Ramsay_MacMullen']
    Magnus Magnusson
    ['/wiki/Magnus_Magnusson']
    Piers Mackesy
    ['/wiki/Piers_Mackesy']
    Leonard Maltin
    ['/wiki/Leonard_Maltin']
    Charles S. Maier
    ['/wiki/Charles_S._Maier']
    Paul L. Maier
    ['/wiki/Paul_L._Maier']
    Pauline Maier
    ['/wiki/Pauline_Maier']
    William Manchester
    ['/wiki/William_Manchester']
    Adel Manna
    ['/wiki/Adel_Manna']
    Golo Mann
    ['/wiki/Golo_Mann']
    Susan Mann
    ['/wiki/Susan_Mann']
    Robert Mann
    ['/wiki/Robert_Mann']
    Philip Mansel
    ['/wiki/Philip_Mansel']
    Arthur Marder
    ['/wiki/Arthur_Marder']
    Timothy Mason
    ['/wiki/Timothy_Mason']
    Henri-Jean Martin
    ['/wiki/Henri-Jean_Martin']
    Rev. F.X. Martin
    ['/wiki/F.X._Martin']
    Michael Marrus
    ['/wiki/Michael_Marrus']
    Mark Mazower
    ['/wiki/Mark_Mazower']
    David McCullough
    ['/wiki/David_McCullough']
    William S. McFeely
    ['/wiki/William_S._McFeely']
    James M. McPherson
    ['/wiki/James_M._McPherson']
    William McNeill
    ['/wiki/William_Hardy_McNeill']
    Laurence Marvin
    ['/wiki/Laurence_Marvin']
    Garrett Mattingly
    ['/wiki/Garrett_Mattingly']
    Arno J. Mayer
    ['/wiki/Arno_J._Mayer']
    Richard Maybury
    ['/wiki/Richard_J._Maybury']
    Neil McKendrick
    ['/wiki/Neil_McKendrick']
    D. W. Meinig
    ['/wiki/D._W._Meinig']
    Evaldo Cabral de Mello
    ['/wiki/Evaldo_Cabral_de_Mello']
    Russell Menard
    ['/wiki/Russell_Menard']
    Thomas C. Mendenhall
    ['/wiki/Thomas_C._Mendenhall_(historian)']
    Josef W. Meri
    ['/wiki/Josef_W._Meri']
    Barbara Metcalf
    ['/wiki/Barbara_Metcalf']
    Rade Mihaljčić
    ['/wiki/Rade_Mihalj%C4%8Di%C4%87']
    Perry Miller
    ['/wiki/Perry_Miller']
    Giles Milton
    ['/wiki/Giles_Milton']
    Zora Mintalová – Zubercová
    ['/wiki/Zora_Mintalov%C3%A1_%E2%80%93_Zubercov%C3%A1']
    Yagutil Mishiev
    ['/wiki/Yagutil_Mishiev']
    Hans Mommsen
    ['/wiki/Hans_Mommsen']
    Wolfgang Mommsen
    ['/wiki/Wolfgang_Mommsen']
    Simon Sebag Montefiore
    ['/wiki/Simon_Sebag_Montefiore']
    Theodore William Moody
    ['/wiki/Theodore_William_Moody']
    Edmund Morgan
    ['/wiki/Edmund_Morgan_(historian)']
    Kenneth O. Morgan
    ['/wiki/Kenneth_O._Morgan']
    William J. Morgan (historian)
    ['/wiki/William_J._Morgan_(historian)']
    Samuel Eliot Morison
    ['/wiki/Samuel_Eliot_Morison']
    Benny Morris
    ['/wiki/Benny_Morris']
    Ian Mortimer
    ['/wiki/Ian_Mortimer_(historian)']
    W.L. Morton
    ['/wiki/W.L._Morton']
    George Mosse
    ['/wiki/George_Mosse']
    Roland Mousnier
    ['/wiki/Roland_Mousnier']
    Mubarak Ali
    ['/wiki/Mubarak_Ali']
    Joseph Needham
    ['/wiki/Joseph_Needham']
    Cynthia Neville
    ['/wiki/Cynthia_Neville']
    Leo Niehorster
    ['/wiki/Leo_Niehorster']
    Thomas Nipperdey
    ['/wiki/Thomas_Nipperdey']
    Ernst Nolte
    ['/wiki/Ernst_Nolte']
    Stojan Novaković
    ['/wiki/Stojan_Novakovi%C4%87']
    Robin O'Neil
    ['/wiki/Robin_O%27Neil']
    Josiah Ober
    ['/wiki/Josiah_Ober']
    Heiko Oberman
    ['/wiki/Heiko_Oberman']
    W. H. Oliver
    ['/wiki/W._H._Oliver']
    Michael Oren
    ['/wiki/Michael_Oren']
    Margaret Ormsby
    ['/wiki/Margaret_Ormsby']
    İlber Ortaylı
    ['/wiki/%C4%B0lber_Ortayl%C4%B1']
    Fernand Ouellet
    ['/wiki/Fernand_Ouellet']
    Richard Overy
    ['/wiki/Richard_Overy']
    Steven Ozment
    ['/wiki/Steven_Ozment']
    Thomas Pakenham
    ['/wiki/Thomas_Pakenham_(historian)']
    Madhavan K. Palat
    ['/wiki/Madhavan_K._Palat']
    Hasan Bülent Paksoy
    ['/wiki/Hasan_B%C3%BClent_Paksoy']
    Ilan Pappé
    ['/wiki/Ilan_Papp%C3%A9']
    Simo Parpola
    ['/wiki/Simo_Parpola']
    J. H. Parry
    ['/wiki/J._H._Parry']
    T. T. Paterson
    ['/wiki/T._T._Paterson']
    Fred Patten
    ['/wiki/Fred_Patten']
    Peter Paret
    ['/wiki/Peter_Paret']
    Geoffrey Parker
    ['/wiki/Geoffrey_Parker_(historian)']
    Stanley G. Payne
    ['/wiki/Stanley_G._Payne']
    Abel Paz
    ['/wiki/Abel_Paz']
    Morgan D. Peoples
    ['/wiki/Morgan_D._Peoples']
    William Armstrong Percy
    ['/wiki/William_Armstrong_Percy']
    Bradford Perkins
    ['/wiki/Bradford_Perkins_(historian)']
    Detlev Peukert
    ['/wiki/Detlev_Peukert']
    Liza Picard
    ['/wiki/Liza_Picard']
    David Pietrusza
    ['/wiki/David_Pietrusza']
    Boris B. Piotrovsky
    ['/wiki/Boris_B._Piotrovsky']
    Richard Pipes
    ['/wiki/Richard_Pipes']
    J.H. Plumb
    ['/wiki/J.H._Plumb']
    J. G. A. Pocock
    ['/wiki/J._G._A._Pocock']
    Kwok Kin Poon
    ['/wiki/Kwok_Kin_Poon']
    Barbara Corrado Pope
    ['/wiki/Barbara_Corrado_Pope']
    Roy Porter
    ['/wiki/Roy_Porter']
    Norman Pounds
    ['/wiki/Norman_Pounds']
    Gordon W. Prange
    ['/wiki/Gordon_W._Prange']
    Joshua Prawer
    ['/wiki/Joshua_Prawer']
    Michael Prestwich
    ['/wiki/Michael_Prestwich']
    Clement Alexander Price
    ['/wiki/Clement_Alexander_Price']
    Francis Paul Prucha
    ['/wiki/Francis_Paul_Prucha']
    Janko Prunk
    ['/wiki/Janko_Prunk']
    Carroll Quigley
    ['/wiki/Carroll_Quigley']
    Marc Raeff
    ['/wiki/Marc_Raeff']
    Werner Rahn
    ['/wiki/Werner_Rahn']
    Jack N. Rakove
    ['/wiki/Jack_N._Rakove']
    Šerbo Rastoder
    ['/wiki/%C5%A0erbo_Rastoder']
    René Rémond
    ['/wiki/Ren%C3%A9_R%C3%A9mond']
    Timothy Reuter
    ['/wiki/Timothy_Reuter']
    Henry A. Reynolds
    ['/wiki/Henry_A._Reynolds']
    Susan Reynolds
    ['/wiki/Susan_Reynolds']
    Richard Rhodes
    ['/wiki/Richard_Rhodes']
    Nicholas V. Riasanovsky
    ['/wiki/Nicholas_V._Riasanovsky']
    Admiral Sir Herbert Richmond
    ['/wiki/Herbert_Richmond']
    Jonathan Riley-Smith
    ['/wiki/Jonathan_Riley-Smith']
    Blaze Ristovski
    ['/wiki/Blaze_Ristovski']
    Charles Ritcheson
    ['/wiki/Charles_Ritcheson']
    Gerhard Ritter
    ['/wiki/Gerhard_Ritter']
    Andrew Roberts
    ['/wiki/Andrew_Roberts_(historian)']
    J. M. Roberts
    ['/wiki/J._M._Roberts']
    N.A.M. Rodger
    ['/wiki/N.A.M._Rodger']
    Walter Rodney
    ['/wiki/Walter_Rodney']
    William Ledyard Rodgers
    ['/wiki/William_Ledyard_Rodgers']
    Theodore Ropp
    ['/wiki/Theodore_Ropp']
    W.J. Rorabaugh
    ['/wiki/W.J._Rorabaugh']
    Ron Rosenbaum
    ['/wiki/Ron_Rosenbaum']
    Charles E. Rosenberg
    ['/wiki/Charles_E._Rosenberg']
    Stephen Roskill
    ['/wiki/Stephen_Roskill']
    Maarten van Rossem
    ['/wiki/Maarten_van_Rossem']
    María Rostworowski
    ['/wiki/Mar%C3%ADa_Rostworowski']
    Theodore Roosevelt
    ['/wiki/Theodore_Roosevelt']
    Michael Rostovtzeff
    ['/wiki/Michael_Rostovtzeff']
    Hans Rothfels
    ['/wiki/Hans_Rothfels']
    Sheila Rowbotham
    ['/wiki/Sheila_Rowbotham']
    Herbert H. Rowen
    ['/wiki/Herbert_H._Rowen']
    A. L. Rowse
    ['/wiki/A._L._Rowse']
    Miri Rubin
    ['/wiki/Miri_Rubin']
    George Rudé
    ['/wiki/George_Rud%C3%A9']
    R. J. Rummel
    ['/wiki/R._J._Rummel']
    Steven Runciman
    ['/wiki/Steven_Runciman']
    Leila J.Rupp
    ['/wiki/Leila_J.Rupp']
    Conrad Russell
    ['/wiki/Conrad_Russell']
    Cornelius Ryan
    ['/wiki/Cornelius_Ryan']
    Boris Rybakov
    ['/wiki/Boris_Rybakov']
    Edgar V. Saks
    ['/wiki/Edgar_V._Saks']
    Richard G. Salomon
    ['/wiki/Richard_G._Salomon']
    S. Srikanta Sastri
    ['/wiki/S._Srikanta_Sastri']
    J. Salwyn Schapiro
    ['/wiki/J._Salwyn_Schapiro']
    Dominic Sandbrook
    ['/wiki/Dominic_Sandbrook']
    Usha Sanyal
    ['/wiki/Usha_Sanyal']
    Simon Schama
    ['/wiki/Simon_Schama']
    Arthur Schlesinger, Sr.
    ['/wiki/Arthur_Schlesinger,_Sr.']
    Arthur Schlesinger, Jr.
    ['/wiki/Arthur_Schlesinger,_Jr.']
    Jean-Claude Schmitt
    ['/wiki/Jean-Claude_Schmitt']
    David Schoenbaum
    ['/wiki/David_Schoenbaum']
    Carl Schorske
    ['/wiki/Carl_Schorske']
    Paul W. Schroeder
    ['/wiki/Paul_W._Schroeder']
    D. M. Schurman
    ['/wiki/D._M._Schurman']
    William Henry Scott
    ['/wiki/William_Henry_Scott_(historian)']
    Joan Scott
    ['/wiki/Joan_Wallach_Scott']
    Howard Hayes Scullard
    ['/wiki/Howard_Hayes_Scullard']
    Oscar Secco Ellauri
    ['/wiki/Oscar_Secco_Ellauri']
    Tom Segev
    ['/wiki/Tom_Segev']
    Robert Service
    ['/wiki/Robert_Service_(historian)']
    Ram Sharan Sharma
    ['/wiki/Ram_Sharan_Sharma']
    James J. Sheehan
    ['/wiki/James_J._Sheehan']
    William L. Shirer
    ['/wiki/William_L._Shirer']
    Dasharatha Sharma
    ['/wiki/Dasharatha_Sharma']
    He Shu
    ['/wiki/He_Shu']
    Jack Simmons
    ['/wiki/Jack_Simmons_(historian)']
    Keith Sinclair
    ['/wiki/Keith_Sinclair']
    Helene J. Sinnreich
    ['/wiki/Helene_J._Sinnreich']
    Nathan Sivin
    ['/wiki/Nathan_Sivin']
    Quentin Skinner
    ['/wiki/Quentin_Skinner']
    Alexandre Skirda
    ['/wiki/Alexandre_Skirda']
    Theda Skocpol
    ['/wiki/Theda_Skocpol']
    Richard Slotkin
    ['/wiki/Richard_Slotkin']
    Cornelius Cole Smith, Jr.
    ['/wiki/Cornelius_Cole_Smith,_Jr.']
    Digby Smith
    ['/wiki/Digby_Smith']
    Henry Nash Smith
    ['/wiki/Henry_Nash_Smith']
    Jean Edward Smith
    ['/wiki/Jean_Edward_Smith']
    Justin Harvey Smith
    ['/wiki/Justin_Harvey_Smith']
    Richard Norton Smith
    ['/wiki/Richard_Norton_Smith']
    T. C. Smout
    ['/wiki/Christopher_Smout']
    Aleksandr Solzhenitsyn
    ['/wiki/Aleksandr_Solzhenitsyn']
    Louis Leo Snyder
    ['/wiki/Louis_Leo_Snyder']
    Timothy D. Snyder
    ['/wiki/Timothy_D._Snyder']
    Albert Soboul
    ['/wiki/Albert_Soboul']
    Pat Southern
    ['/wiki/Pat_Southern']
    Richard Southern
    ['/wiki/Richard_Southern']
    Dr. E. Lee Spence
    ['/wiki/Dr._E._Lee_Spence']
    Jonathan Spence
    ['/wiki/Jonathan_Spence']
    Jackson J. Spielvogel
    ['/wiki/Jackson_J._Spielvogel']
    Kenneth Stampp
    ['/wiki/Kenneth_Stampp']
    George Stanley
    ['/wiki/George_Stanley']
    Stanoje Stanojević
    ['/wiki/Stanoje_Stanojevi%C4%87']
    David Starkey
    ['/wiki/David_Starkey']
    Leften Stavros Stavrianos
    ['/wiki/Leften_Stavros_Stavrianos']
    James M. Stayer
    ['/wiki/James_M._Stayer']
    Wickham Steed
    ['/wiki/Wickham_Steed']
    Valerie Steele
    ['/wiki/Valerie_Steele']
    Jean Stengers
    ['/wiki/Jean_Stengers']
    Frank Stenton
    ['/wiki/Frank_Stenton']
    Fritz Stern
    ['/wiki/Fritz_Stern']
    Zeev Sternhell
    ['/wiki/Zeev_Sternhell']
    William N. Still, Jr.
    ['/wiki/William_N._Still,_Jr.']
    Lawrence Stone
    ['/wiki/Lawrence_Stone']
    Norman Stone
    ['/wiki/Norman_Stone']
    Hew Strachan
    ['/wiki/Hew_Strachan']
    Barry S. Strauss
    ['/wiki/Barry_S._Strauss']
    Floyd Benjamin Streeter
    ['/wiki/Floyd_Benjamin_Streeter']
    Michael Stürmer
    ['/wiki/Michael_St%C3%BCrmer']
    Ronald Suleski
    ['/wiki/Ronald_Suleski']
    Viktor Suvorov
    ['/wiki/Viktor_Suvorov']
    David Syrett
    ['/wiki/David_Syrett']
    Ronald Syme
    ['/wiki/Ronald_Syme']
    J. L. Talmon
    ['/wiki/J._L._Talmon']
    A.J.P. Taylor
    ['/wiki/A.J.P._Taylor']
    Alasdair and Hettie Tayler
    ['/wiki/Alasdair_and_Hettie_Tayler']
    Ronald Takaki
    ['/wiki/Ronald_Takaki']
    Abdelhadi Tazi
    ['/wiki/Abdelhadi_Tazi']
    Antonio Tellez
    ['/wiki/Antonio_Tellez']
    Harold Temperley
    ['/wiki/Harold_Temperley']
    Romila Thapar
    ['/wiki/Romila_Thapar']
    Barbara Thiering
    ['/wiki/Barbara_Thiering']
    Joan Thirsk
    ['/wiki/Joan_Thirsk']
    Hugh Thomas
    ['/wiki/Hugh_Thomas_(historian)']
    E. P. Thompson
    ['/wiki/E._P._Thompson']
    John Toland
    ['/wiki/John_Toland_(author)']
    K. Ross Toole
    ['/wiki/K._Ross_Toole']
    Ahmed Toufiq
    ['/wiki/Ahmed_Toufiq']
    Marc Trachtenberg
    ['/wiki/Marc_Trachtenberg']
    Hugh Trevor-Roper
    ['/wiki/Hugh_Trevor-Roper']
    Gil Troy
    ['/wiki/Gil_Troy']
    Barbara Tuchman
    ['/wiki/Barbara_Tuchman']
    Robert C. Tucker
    ['/wiki/Robert_C._Tucker']
    Peter Turchin
    ['/wiki/Peter_Turchin']
    Henry Ashby Turner, Jr.
    ['/wiki/Henry_Ashby_Turner,_Jr.']
    Frederick Jackson Turner
    ['/wiki/Frederick_Jackson_Turner']
    Denis Twitchett
    ['/wiki/Denis_Twitchett']
    David Tyack
    ['/wiki/David_Tyack']
    Walter Ullmann
    ['/wiki/Walter_Ullmann']
    Laurel Thatcher Ulrich
    ['/wiki/Laurel_Thatcher_Ulrich']
    Mladen Urem
    ['/wiki/Mladen_Urem']
    Robert M. Utley
    ['/wiki/Robert_M._Utley']
    Hans van de Ven
    ['/wiki/Hans_van_de_Ven']
    Jean-Pierre Vernant
    ['/wiki/Jean-Pierre_Vernant']
    Paul Veyne
    ['/wiki/Paul_Veyne']
    César Vidal Manzanares
    ['/wiki/C%C3%A9sar_Vidal_Manzanares']
    Pierre Vidal-Naquet
    ['/wiki/Pierre_Vidal-Naquet']
    Richard Vinen
    ['/wiki/Richard_Vinen']
    Klemens von Klemperer
    ['/wiki/Klemens_von_Klemperer']
    William Dalrymple
    ['/wiki/William_Dalrymple_(historian)']
    John Waiko
    ['/wiki/John_Waiko']
    J. Samuel Walker
    ['/wiki/J._Samuel_Walker']
    Retha Warnicke
    ['/wiki/Retha_Warnicke']
    Eugen Weber
    ['/wiki/Eugen_Weber']
    Cicely Veronica Wedgwood
    ['/wiki/Cicely_Veronica_Wedgwood']
    Hans-Ulrich Wehler
    ['/wiki/Hans-Ulrich_Wehler']
    Russell Weigley
    ['/wiki/Russell_Weigley']
    Gerhard Weinberg
    ['/wiki/Gerhard_Weinberg']
    Roberto Weiss
    ['/wiki/Roberto_Weiss']
    Frank Welsh
    ['/wiki/Frank_Welsh_(writer)']
    Christopher Whatley
    ['/wiki/Christopher_Whatley']
    John Wheeler-Bennett
    ['/wiki/John_Wheeler-Bennett']
    John Whyte
    ['/wiki/John_Henry_Whyte']
    Christopher Wickham
    ['/wiki/Christopher_Wickham']
    Alexander Wilkinson
    ['/wiki/Alexander_Wilkinson']
    Toby Wilkinson
    ['/wiki/Toby_Wilkinson']
    Eric Williams
    ['/wiki/Eric_Williams']
    Glanmor Williams
    ['/wiki/Glanmor_Williams']
    Glyndwr Williams
    ['/wiki/Glyndwr_Williams']
    William Appleman Williams
    ['/wiki/William_Appleman_Williams']
    John Willingham
    ['/wiki/John_Willingham']
    Andrew Wilson
    ['/wiki/Andrew_Wilson_(historian)']
    Clyde N. Wilson
    ['/wiki/Clyde_N._Wilson']
    Ian Wilson
    ['/wiki/Ian_Wilson_(writer)']
    Henry Winkler
    ['/wiki/Heinrich_August_Winkler']
    Keith Windschuttle
    ['/wiki/Keith_Windschuttle']
    Gordon Wright
    ['/wiki/Gordon_Wright_(historian)']
    Robert S. Wistrich
    ['/wiki/Robert_S._Wistrich']
    John B. Wolf
    ['/wiki/John_Baptist_Wolf']
    Michael Wolffsohn
    ['/wiki/Michael_Wolffsohn']
    Herwig Wolfram
    ['/wiki/Herwig_Wolfram']
    Gordon S. Wood
    ['/wiki/Gordon_S._Wood']
    Michael Wood
    ['/wiki/Michael_Wood_(historian)']
    Thomas Woods
    ['/wiki/Thomas_Woods']
    C. Vann Woodward
    ['/wiki/C._Vann_Woodward']
    Lucy Worsley
    ['/wiki/Lucy_Worsley']
    Lawrence C. Wroth
    ['/wiki/Lawrence_C._Wroth']
    Robert J. Young
    ['/wiki/Robert_J._Young']
    Robert M. Young
    ['/wiki/Robert_M._Young_(academic)']
    Nicolas Zafra
    ['/wiki/Nicolas_Zafra']
    Gregorio F. Zaide
    ['/wiki/Gregorio_F._Zaide']
    Adam Zamoyski
    ['/wiki/Adam_Zamoyski']
    Alfred-Maurice de Zayas
    ['/wiki/Alfred-Maurice_de_Zayas']
    Howard Zinn
    ['/wiki/Howard_Zinn']
    Rainer Zitelmann
    ['/wiki/Rainer_Zitelmann']
    Marek Żukow-Karczewski
    ['/wiki/Marek_%C5%BBukow-Karczewski']
    Historiography
    ['/wiki/Historiography']
    History
    ['/wiki/History']
    List of historians by area of study
    ['/wiki/List_of_historians_by_area_of_study']
    List of Canadian historians
    ['/wiki/List_of_Canadian_historians']
    List of history journals
    ['/wiki/List_of_history_journals']
    List of Irish historians
    ['/wiki/List_of_Irish_historians']
    List of Jewish historians
    ['/wiki/List_of_Jewish_historians']
    List of Russian historians
    ['/wiki/List_of_Russian_historians']
    Historian
    ['/wiki/Historian']
    Momigliano, Arnaldo
    ['/wiki/Arnaldo_Momigliano']


At this point I've already extracted a bunch of useful information from a webpage! Let's step through. 
1. create an empty list for the data I want to save, called historians
2. the xpath here leads to all the list elements, and allows us to cycle through each one
    1. for each element, we can apply xpath that applys only within that element
    2. We print, for fun, and to make sure we're doing what we intend. 
        1. The name is just saved in the text of the tag, and .text gets the text for us! 
        2. The link is in the attribute href, notably the @href (link) tag comes back inside a list with *one* element **and** it comes back as a stub
    3. Save the info to a dictionary that we'll keep in a list
        1. name is under 'name'
        2. we add the root to the stub, and we index at [0] because the tag returns a list w/ just one value


```python
historians 
```




    [{'name': 'Herodotus', 'url': 'https://en.wikipedia.org/wiki/Herodotus'},
     {'name': 'Thucydides', 'url': 'https://en.wikipedia.org/wiki/Thucydides'},
     {'name': 'Xenophon', 'url': 'https://en.wikipedia.org/wiki/Xenophon'},
     {'name': 'Ctesias', 'url': 'https://en.wikipedia.org/wiki/Ctesias'},
     {'name': 'Theopompus', 'url': 'https://en.wikipedia.org/wiki/Theopompus'},
     {'name': 'Eudemus of Rhodes',
      'url': 'https://en.wikipedia.org/wiki/Eudemus_of_Rhodes'},
     {'name': 'Berossus', 'url': 'https://en.wikipedia.org/wiki/Berossus'},
     {'name': 'Ptolemy I Soter',
      'url': 'https://en.wikipedia.org/wiki/Ptolemy_I_Soter'},
     {'name': 'Duris of Samos',
      'url': 'https://en.wikipedia.org/wiki/Duris_of_Samos'},
     {'name': 'Manetho', 'url': 'https://en.wikipedia.org/wiki/Manetho'},
     {'name': 'Timaeus of Tauromenium',
      'url': 'https://en.wikipedia.org/wiki/Timaeus_(historian)'},
     {'name': 'Quintus Fabius Pictor',
      'url': 'https://en.wikipedia.org/wiki/Quintus_Fabius_Pictor'},
     {'name': 'Artapanus of Alexandria',
      'url': 'https://en.wikipedia.org/wiki/Artapanus_of_Alexandria'},
     {'name': 'Cato the Elder',
      'url': 'https://en.wikipedia.org/wiki/Cato_the_Elder'},
     {'name': 'Gaius Acilius',
      'url': 'https://en.wikipedia.org/wiki/Gaius_Acilius'},
     {'name': 'Polybius', 'url': 'https://en.wikipedia.org/wiki/Polybius'},
     {'name': 'Sempronius Asellio',
      'url': 'https://en.wikipedia.org/wiki/Sempronius_Asellio'},
     {'name': 'Sima Tan', 'url': 'https://en.wikipedia.org/wiki/Sima_Tan'},
     {'name': 'Sima Qian', 'url': 'https://en.wikipedia.org/wiki/Sima_Qian'},
     {'name': 'Agatharchides',
      'url': 'https://en.wikipedia.org/wiki/Agatharchides'},
     {'name': 'Posidonius', 'url': 'https://en.wikipedia.org/wiki/Posidonius'},
     {'name': 'Julius Caesar',
      'url': 'https://en.wikipedia.org/wiki/Julius_Caesar'},
     {'name': 'Diodorus of Sicily',
      'url': 'https://en.wikipedia.org/wiki/Diodorus_Siculus'},
     {'name': 'Sallust', 'url': 'https://en.wikipedia.org/wiki/Sallust'},
     {'name': 'Liu Xiang (scholar)',
      'url': 'https://en.wikipedia.org/wiki/Liu_Xiang_(scholar)'},
     {'name': 'Theophanes of Mytilene',
      'url': 'https://en.wikipedia.org/wiki/Theophanes_of_Mytilene'},
     {'name': 'Dionysius of Halicarnassus',
      'url': 'https://en.wikipedia.org/wiki/Dionysius_of_Halicarnassus'},
     {'name': 'Strabo', 'url': 'https://en.wikipedia.org/wiki/Strabo'},
     {'name': 'Livy', 'url': 'https://en.wikipedia.org/wiki/Livy'},
     {'name': 'Marcus Velleius Paterculus',
      'url': 'https://en.wikipedia.org/wiki/Marcus_Velleius_Paterculus'},
     {'name': 'Memnon of Heraclea',
      'url': 'https://en.wikipedia.org/wiki/Memnon_of_Heraclea'},
     {'name': 'Ban Biao', 'url': 'https://en.wikipedia.org/wiki/Ban_Biao'},
     {'name': 'Quintus Curtius Rufus',
      'url': 'https://en.wikipedia.org/wiki/Quintus_Curtius_Rufus'},
     {'name': 'Ban Gu', 'url': 'https://en.wikipedia.org/wiki/Ban_Gu'},
     {'name': 'Flavius Josephus', 'url': 'https://en.wikipedia.org/wiki/Josephus'},
     {'name': 'Pamphile of Epidaurus',
      'url': 'https://en.wikipedia.org/wiki/Pamphile_of_Epidaurus'},
     {'name': 'Ban Zhao', 'url': 'https://en.wikipedia.org/wiki/Ban_Zhao'},
     {'name': 'Thallus',
      'url': 'https://en.wikipedia.org/wiki/Thallus_(historian)'},
     {'name': 'Plutarch', 'url': 'https://en.wikipedia.org/wiki/Plutarch'},
     {'name': 'Gaius Cornelius Tacitus',
      'url': 'https://en.wikipedia.org/wiki/Gaius_Cornelius_Tacitus'},
     {'name': 'Suetonius',
      'url': 'https://en.wikipedia.org/wiki/Lives_of_the_Twelve_Caesars'},
     {'name': 'Appian', 'url': 'https://en.wikipedia.org/wiki/Appian'},
     {'name': 'Arrian', 'url': 'https://en.wikipedia.org/wiki/Arrian'},
     {'name': 'Lucius Ampelius',
      'url': 'https://en.wikipedia.org/wiki/Liber_Memorialis'},
     {'name': 'Dio Cassius', 'url': 'https://en.wikipedia.org/wiki/Dio_Cassius'},
     {'name': 'Herodian', 'url': 'https://en.wikipedia.org/wiki/Herodian'},
     {'name': 'Sextus Julius Africanus',
      'url': 'https://en.wikipedia.org/wiki/Sextus_Julius_Africanus'},
     {'name': u'Diogenes La\xebrtius',
      'url': 'https://en.wikipedia.org/wiki/Diogenes_La%C3%ABrtius'},
     {'name': 'Chen Shou', 'url': 'https://en.wikipedia.org/wiki/Chen_Shou'},
     {'name': 'Eusebius of Caesarea',
      'url': 'https://en.wikipedia.org/wiki/Eusebius_of_Caesarea'},
     {'name': 'Ammianus Marcellinus',
      'url': 'https://en.wikipedia.org/wiki/Ammianus_Marcellinus'},
     {'name': 'Fa-Hien', 'url': 'https://en.wikipedia.org/wiki/Fa-Hien'},
     {'name': 'Rufinus of Aquileia',
      'url': 'https://en.wikipedia.org/wiki/Rufinus_of_Aquileia'},
     {'name': 'Philostorgius',
      'url': 'https://en.wikipedia.org/wiki/Philostorgius'},
     {'name': 'Socrates of Constantinople',
      'url': 'https://en.wikipedia.org/wiki/Socrates_of_Constantinople'},
     {'name': 'Theodoret', 'url': 'https://en.wikipedia.org/wiki/Theodoret'},
     {'name': 'Fan Ye (historian)',
      'url': 'https://en.wikipedia.org/wiki/Fan_Ye_(historian)'},
     {'name': 'Priscus', 'url': 'https://en.wikipedia.org/wiki/Priscus'},
     {'name': 'Sozomen', 'url': 'https://en.wikipedia.org/wiki/Sozomen'},
     {'name': 'Salvian', 'url': 'https://en.wikipedia.org/wiki/Salvian'},
     {'name': 'Movses Khorenatsi',
      'url': 'https://en.wikipedia.org/wiki/Movses_Khorenatsi'},
     {'name': 'Shen Yue', 'url': 'https://en.wikipedia.org/wiki/Shen_Yue'},
     {'name': 'John Malalas', 'url': 'https://en.wikipedia.org/wiki/John_Malalas'},
     {'name': 'Zosimus', 'url': 'https://en.wikipedia.org/wiki/Zosimus'},
     {'name': 'Procopius', 'url': 'https://en.wikipedia.org/wiki/Procopius'},
     {'name': 'Jordanes', 'url': 'https://en.wikipedia.org/wiki/Jordanes'},
     {'name': 'Gregory of Tours',
      'url': 'https://en.wikipedia.org/wiki/Gregory_of_Tours'},
     {'name': 'Wei Zheng', 'url': 'https://en.wikipedia.org/wiki/Wei_Zheng'},
     {'name': 'Baudovinia', 'url': 'https://en.wikipedia.org/wiki/Baudovinia'},
     {'name': 'Yao Silian', 'url': 'https://en.wikipedia.org/wiki/Yao_Silian'},
     {'name': 'Fang Xuanling',
      'url': 'https://en.wikipedia.org/wiki/Fang_Xuanling'},
     {'name': 'Adamnan', 'url': 'https://en.wikipedia.org/wiki/Adamnan'},
     {'name': 'Bede', 'url': 'https://en.wikipedia.org/wiki/Bede'},
     {'name': u'T\xedrech\xe1n',
      'url': 'https://en.wikipedia.org/wiki/T%C3%ADrech%C3%A1n'},
     {'name': 'Cogitosus', 'url': 'https://en.wikipedia.org/wiki/Cogitosus'},
     {'name': 'Muirchu moccu Machtheni',
      'url': 'https://en.wikipedia.org/wiki/Muirchu_moccu_Machtheni'},
     {'name': 'Paul the Deacon',
      'url': 'https://en.wikipedia.org/wiki/Paul_the_Deacon'},
     {'name': 'Liu Zhiji', 'url': 'https://en.wikipedia.org/wiki/Liu_Zhiji'},
     {'name': u'\u014c no Yasumaro',
      'url': 'https://en.wikipedia.org/wiki/%C5%8C_no_Yasumaro'},
     {'name': 'Constantine of Preslav',
      'url': 'https://en.wikipedia.org/wiki/Constantine_of_Preslav'},
     {'name': 'Nennius', 'url': 'https://en.wikipedia.org/wiki/Nennius'},
     {'name': 'Martianus Hiberniensis',
      'url': 'https://en.wikipedia.org/wiki/Martianus_Hiberniensis'},
     {'name': 'Einhard', 'url': 'https://en.wikipedia.org/wiki/Einhard'},
     {'name': 'Notker of St Gall',
      'url': 'https://en.wikipedia.org/wiki/Notker_of_St_Gall'},
     {'name': u'Regino of Pr\xfcm',
      'url': 'https://en.wikipedia.org/wiki/Regino_of_Pr%C3%BCm'},
     {'name': 'Asser', 'url': 'https://en.wikipedia.org/wiki/Asser'},
     {'name': 'Muhammad al-Tabari',
      'url': 'https://en.wikipedia.org/wiki/Muhammad_ibn_Jarir_al-Tabari'},
     {'name': 'Liu Xu', 'url': 'https://en.wikipedia.org/wiki/Liu_Xu'},
     {'name': 'Liutprand of Cremona',
      'url': 'https://en.wikipedia.org/wiki/Liutprand_of_Cremona'},
     {'name': 'Li Fang',
      'url': 'https://en.wikipedia.org/wiki/Li_Fang_(Song_dynasty)'},
     {'name': 'Heriger of Lobbes',
      'url': 'https://en.wikipedia.org/wiki/Heriger_of_Lobbes'},
     {'name': 'Al-Biruni', 'url': 'https://en.wikipedia.org/wiki/Al-Biruni'},
     {'name': 'Thietmar of Merseburg',
      'url': 'https://en.wikipedia.org/wiki/Thietmar_of_Merseburg'},
     {'name': 'Ibn Rustah', 'url': 'https://en.wikipedia.org/wiki/Ibn_Rustah'},
     {'name': 'Song Qi', 'url': 'https://en.wikipedia.org/wiki/Song_Qi'},
     {'name': 'Ouyang Xiu', 'url': 'https://en.wikipedia.org/wiki/Ouyang_Xiu'},
     {'name': 'Albert of Aix',
      'url': 'https://en.wikipedia.org/wiki/Albert_of_Aix'},
     {'name': 'Nestor the Chronicler',
      'url': 'https://en.wikipedia.org/wiki/Nestor_the_Chronicler'},
     {'name': 'Gallus Anonymus',
      'url': 'https://en.wikipedia.org/wiki/Gallus_Anonymus'},
     {'name': 'Michael Attaleiates',
      'url': 'https://en.wikipedia.org/wiki/Michael_Attaleiates'},
     {'name': 'Michael Psellus',
      'url': 'https://en.wikipedia.org/wiki/Michael_Psellus'},
     {'name': 'Sima Guang', 'url': 'https://en.wikipedia.org/wiki/Sima_Guang'},
     {'name': 'Marianus Scotus',
      'url': 'https://en.wikipedia.org/wiki/Marianus_Scotus'},
     {'name': 'Guibert of Nogent',
      'url': 'https://en.wikipedia.org/wiki/Guibert_of_Nogent'},
     {'name': 'Adam of Bremen',
      'url': 'https://en.wikipedia.org/wiki/Adam_of_Bremen'},
     {'name': 'Galbert of Bruges',
      'url': 'https://en.wikipedia.org/wiki/Galbert_of_Bruges'},
     {'name': 'Florence of Worcester',
      'url': 'https://en.wikipedia.org/wiki/Florence_of_Worcester'},
     {'name': 'Eadmer', 'url': 'https://en.wikipedia.org/wiki/Eadmer'},
     {'name': 'Kim Bu-sik', 'url': 'https://en.wikipedia.org/wiki/Kim_Bu-sik'},
     {'name': 'William of Malmesbury',
      'url': 'https://en.wikipedia.org/wiki/William_of_Malmesbury'},
     {'name': 'Symeon of Durham',
      'url': 'https://en.wikipedia.org/wiki/Symeon_of_Durham'},
     {'name': 'Anna Comnena', 'url': 'https://en.wikipedia.org/wiki/Anna_Comnena'},
     {'name': 'Usamah ibn Munqidh',
      'url': 'https://en.wikipedia.org/wiki/Usamah_ibn_Munqidh'},
     {'name': 'Geoffrey of Monmouth',
      'url': 'https://en.wikipedia.org/wiki/Geoffrey_of_Monmouth'},
     {'name': 'Helmold of Bosau',
      'url': 'https://en.wikipedia.org/wiki/Helmold_of_Bosau'},
     {'name': 'William of Tyre',
      'url': 'https://en.wikipedia.org/wiki/William_of_Tyre'},
     {'name': 'Alured of Beverley',
      'url': 'https://en.wikipedia.org/wiki/Alured_of_Beverley'},
     {'name': 'William of Newburgh',
      'url': 'https://en.wikipedia.org/wiki/William_of_Newburgh'},
     {'name': 'Svend Aagesen',
      'url': 'https://en.wikipedia.org/wiki/Svend_Aagesen'},
     {'name': 'Mohammed al-Baydhaq',
      'url': 'https://en.wikipedia.org/wiki/Mohammed_al-Baydhaq'},
     {'name': 'John of Worcester',
      'url': 'https://en.wikipedia.org/wiki/John_of_Worcester'},
     {'name': 'Giraldus Cambrensis',
      'url': 'https://en.wikipedia.org/wiki/Giraldus_Cambrensis'},
     {'name': 'Wincenty Kadlubek',
      'url': 'https://en.wikipedia.org/wiki/Wincenty_Kadlubek'},
     {'name': 'Ambroise', 'url': 'https://en.wikipedia.org/wiki/Ambroise'},
     {'name': 'Geoffroi de Villehardouin',
      'url': 'https://en.wikipedia.org/wiki/Geoffroi_de_Villehardouin'},
     {'name': 'Kalhana', 'url': 'https://en.wikipedia.org/wiki/Kalhana'},
     {'name': 'Saxo Grammaticus',
      'url': 'https://en.wikipedia.org/wiki/Saxo_Grammaticus'},
     {'name': 'Joannes Zonaras',
      'url': 'https://en.wikipedia.org/wiki/Joannes_Zonaras'},
     {'name': 'Nicetas Choniates',
      'url': 'https://en.wikipedia.org/wiki/Nicetas_Choniates'},
     {'name': 'Snorri Sturluson',
      'url': 'https://en.wikipedia.org/wiki/Snorri_Sturluson'},
     {'name': 'Abdelwahid al-Marrakushi',
      'url': 'https://en.wikipedia.org/wiki/Abdelwahid_al-Marrakushi'},
     {'name': 'Ata al-Mulk Juvayni',
      'url': 'https://en.wikipedia.org/wiki/Ata_al-Mulk_Juvayni'},
     {'name': 'Ibn al-Khabbaza',
      'url': 'https://en.wikipedia.org/wiki/Ibn_al-Khabbaza'},
     {'name': 'Matthew Paris',
      'url': 'https://en.wikipedia.org/wiki/Matthew_Paris'},
     {'name': 'Domentijan', 'url': 'https://en.wikipedia.org/wiki/Domentijan'},
     {'name': 'Il-yeon', 'url': 'https://en.wikipedia.org/wiki/Il-yeon'},
     {'name': 'Salimbene di Adam',
      'url': 'https://en.wikipedia.org/wiki/Salimbene_di_Adam'},
     {'name': 'Abdelaziz al-Malzuzi',
      'url': 'https://en.wikipedia.org/wiki/Abdelaziz_al-Malzuzi'},
     {'name': 'Templar of Tyre',
      'url': 'https://en.wikipedia.org/wiki/Templar_of_Tyre'},
     {'name': u'L\xea V\u0103n H\u01b0u',
      'url': 'https://en.wikipedia.org/wiki/L%C3%AA_V%C4%83n_H%C6%B0u'},
     {'name': 'Adam of Eynsham',
      'url': 'https://en.wikipedia.org/wiki/Adam_of_Eynsham'},
     {'name': 'Jean de Joinville',
      'url': 'https://en.wikipedia.org/wiki/Jean_de_Joinville'},
     {'name': 'Piers Langtoft',
      'url': 'https://en.wikipedia.org/wiki/Piers_Langtoft'},
     {'name': 'Rashid-al-Din Hamadani',
      'url': 'https://en.wikipedia.org/wiki/Rashid-al-Din_Hamadani'},
     {'name': 'Giovanni Villani',
      'url': 'https://en.wikipedia.org/wiki/Giovanni_Villani'},
     {'name': 'Ibn Idhari', 'url': 'https://en.wikipedia.org/wiki/Ibn_Idhari'},
     {'name': 'Ibn Abi Zar', 'url': 'https://en.wikipedia.org/wiki/Ibn_Abi_Zar'},
     {'name': 'Abdullah Wassaf', 'url': 'https://en.wikipedia.org/wiki/Wassaf'},
     {'name': 'Song Lian', 'url': 'https://en.wikipedia.org/wiki/Song_Lian'},
     {'name': "Toqto'a",
      'url': 'https://en.wikipedia.org/wiki/Toqto%27a_(Yuan_Dynasty)'},
     {'name': 'ibn Khaldun', 'url': 'https://en.wikipedia.org/wiki/Ibn_Khaldun'},
     {'name': 'John Clyn', 'url': 'https://en.wikipedia.org/wiki/John_Clyn'},
     {'name': 'Baldassarre Bonaiuti',
      'url': 'https://en.wikipedia.org/wiki/Baldassarre_Bonaiuti'},
     {'name': 'Jean Froissart',
      'url': 'https://en.wikipedia.org/wiki/Jean_Froissart'},
     {'name': 'Dietrich of Nieheim',
      'url': 'https://en.wikipedia.org/wiki/Dietrich_of_Nieheim'},
     {'name': 'John of Fordun',
      'url': 'https://en.wikipedia.org/wiki/John_of_Fordun'},
     {'name': u'Ruaidhri \xd3 Cian\xe1in',
      'url': 'https://en.wikipedia.org/wiki/Ruaidhri_%C3%93_Cian%C3%A1in'},
     {'name': 'Christine de Pizan',
      'url': 'https://en.wikipedia.org/wiki/Christine_de_Pizan'},
     {'name': u'\xc1lvar Garc\xeda de Santa Mar\xeda',
      'url': 'https://en.wikipedia.org/wiki/%C3%81lvar_Garc%C3%ADa_de_Santa_Mar%C3%ADa'},
     {'name': u'Se\xe1n M\xf3r \xd3 Dubhag\xe1in',
      'url': 'https://en.wikipedia.org/wiki/Se%C3%A1n_M%C3%B3r_%C3%93_Dubhag%C3%A1in'},
     {'name': u'Adhamh \xd3 Cian\xe1in',
      'url': 'https://en.wikipedia.org/wiki/Adhamh_%C3%93_Cian%C3%A1in'},
     {'name': 'Ismail ibn al-Ahmar',
      'url': 'https://en.wikipedia.org/wiki/Ismail_ibn_al-Ahmar'},
     {'name': u'Giolla \xcdosa M\xf3r Mac Fhirbhisigh',
      'url': 'https://en.wikipedia.org/wiki/Giolla_%C3%8Dosa_M%C3%B3r_Mac_Fhirbhisigh'},
     {'name': 'Zhu Quan', 'url': 'https://en.wikipedia.org/wiki/Zhu_Quan'},
     {'name': 'John Capgrave',
      'url': 'https://en.wikipedia.org/wiki/John_Capgrave'},
     {'name': 'Alphonsus A Sancta Maria',
      'url': 'https://en.wikipedia.org/wiki/Alphonsus_A_Sancta_Maria'},
     {'name': u'Jan D\u0142ugosz',
      'url': 'https://en.wikipedia.org/wiki/Jan_D%C5%82ugosz'},
     {'name': 'Sharaf ad-Din Ali Yazdi',
      'url': 'https://en.wikipedia.org/wiki/Sharaf_ad-Din_Ali_Yazdi'},
     {'name': u'Cathal \xd3g Mac Maghnusa',
      'url': 'https://en.wikipedia.org/wiki/Cathal_%C3%93g_Mac_Maghnusa'},
     {'name': 'Philippe de Commines',
      'url': 'https://en.wikipedia.org/wiki/Philippe_de_Commines'},
     {'name': 'Robert Fabyan',
      'url': 'https://en.wikipedia.org/wiki/Robert_Fabyan'},
     {'name': 'Albert Krantz',
      'url': 'https://en.wikipedia.org/wiki/Albert_Krantz'},
     {'name': 'Hector Boece', 'url': 'https://en.wikipedia.org/wiki/Hector_Boece'},
     {'name': 'Polydore Vergil',
      'url': 'https://en.wikipedia.org/wiki/Polydore_Vergil'},
     {'name': 'Sigismund von Herberstein',
      'url': 'https://en.wikipedia.org/wiki/Sigismund_von_Herberstein'},
     {'name': u'Jo\xe3o de Barros',
      'url': 'https://en.wikipedia.org/wiki/Jo%C3%A3o_de_Barros'},
     {'name': u'Niccol\xf2 Machiavelli',
      'url': 'https://en.wikipedia.org/wiki/Niccol%C3%B2_Machiavelli'},
     {'name': 'Francesco Guicciardini',
      'url': 'https://en.wikipedia.org/wiki/Francesco_Guicciardini'},
     {'name': 'Josias Simmler',
      'url': 'https://en.wikipedia.org/wiki/Josias_Simmler'},
     {'name': 'Paolo Paruta', 'url': 'https://en.wikipedia.org/wiki/Paolo_Paruta'},
     {'name': 'Raphael Holinshed',
      'url': 'https://en.wikipedia.org/wiki/Raphael_Holinshed'},
     {'name': 'Caesar Baronius',
      'url': 'https://en.wikipedia.org/wiki/Caesar_Baronius'},
     {'name': "Abd al-Qadir Bada'uni",
      'url': 'https://en.wikipedia.org/wiki/Abd_al-Qadir_Bada%27uni'},
     {'name': 'Abd al-Aziz al-Fishtali',
      'url': 'https://en.wikipedia.org/wiki/Abd_al-Aziz_al-Fishtali'},
     {'name': 'Ahmad Ibn al-Qadi',
      'url': 'https://en.wikipedia.org/wiki/Ahmad_Ibn_al-Qadi'},
     {'name': 'John Hayward',
      'url': 'https://en.wikipedia.org/wiki/John_Hayward_(historian)'},
     {'name': u'Pilip Ballach \xd3 Duibhgeann\xe1in',
      'url': 'https://en.wikipedia.org/wiki/Pilip_Ballach_%C3%93_Duibhgeann%C3%A1in'},
     {'name': 'James Ussher', 'url': 'https://en.wikipedia.org/wiki/James_Ussher'},
     {'name': 'William Bradford',
      'url': 'https://en.wikipedia.org/wiki/William_Bradford_(Plymouth_governor)'},
     {'name': 'Bahrey', 'url': 'https://en.wikipedia.org/wiki/Bahrey'},
     {'name': u'Fray \xcd\xf1igo Abbad y Lasierra',
      'url': 'https://en.wikipedia.org/wiki/Fray_%C3%8D%C3%B1igo_Abbad_y_Lasierra'},
     {'name': 'Mohammed Akensus',
      'url': 'https://en.wikipedia.org/wiki/Mohammed_Akensus'},
     {'name': 'Archibald Alison',
      'url': 'https://en.wikipedia.org/wiki/Sir_Archibald_Alison,_1st_Baronet'},
     {'name': 'Abbasgulu Bakikhanov',
      'url': 'https://en.wikipedia.org/wiki/Abbasgulu_Bakikhanov'},
     {'name': 'Teimuraz Bagrationi',
      'url': 'https://en.wikipedia.org/wiki/Teimuraz_Bagrationi'},
     {'name': 'Archibald Bower',
      'url': 'https://en.wikipedia.org/wiki/Archibald_Bower'},
     {'name': u'\u0110or\u0111e Brankovi\u0107',
      'url': 'https://en.wikipedia.org/wiki/%C4%90or%C4%91e_Brankovi%C4%87_(count)'},
     {'name': 'Mary Bonaventure Browne',
      'url': 'https://en.wikipedia.org/wiki/Mary_Bonaventure_Browne'},
     {'name': 'Josiah Burchett',
      'url': 'https://en.wikipedia.org/wiki/Josiah_Burchett'},
     {'name': 'Ron Chernow', 'url': 'https://en.wikipedia.org/wiki/Ron_Chernow'},
     {'name': u"Chang Hs\xfceh-ch'eng",
      'url': 'https://en.wikipedia.org/wiki/Chang_Hs%C3%BCeh-ch%27eng'},
     {'name': 'Thomas Carlyle',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Carlyle'},
     {'name': 'Chaudhry Afzal Haq',
      'url': 'https://en.wikipedia.org/wiki/Chaudhry_Afzal_Haq'},
     {'name': 'Agha Shorish Kashmiri',
      'url': 'https://en.wikipedia.org/wiki/Agha_Shorish_Kashmiri'},
     {'name': 'Janbaz Mirza', 'url': 'https://en.wikipedia.org/wiki/Janbaz_Mirza'},
     {'name': 'Simonas Daukantas',
      'url': 'https://en.wikipedia.org/wiki/Simonas_Daukantas'},
     {'name': 'Charles Dezobry',
      'url': 'https://en.wikipedia.org/wiki/Charles_Dezobry'},
     {'name': 'Mohammed al-Duayf',
      'url': 'https://en.wikipedia.org/wiki/Mohammed_al-Duayf'},
     {'name': 'John Colin Dunlop',
      'url': 'https://en.wikipedia.org/wiki/John_Colin_Dunlop'},
     {'name': 'Laurence Echard',
      'url': 'https://en.wikipedia.org/wiki/Laurence_Echard'},
     {'name': 'Abd al-Rahman al-Fasi',
      'url': 'https://en.wikipedia.org/wiki/Abd_al-Rahman_al-Fasi'},
     {'name': 'George Finlay',
      'url': 'https://en.wikipedia.org/wiki/George_Finlay'},
     {'name': 'Abd al-Aziz al-Fishtali',
      'url': 'https://en.wikipedia.org/wiki/Abd_al-Aziz_al-Fishtali'},
     {'name': 'Francisco Jose Freire',
      'url': 'https://en.wikipedia.org/wiki/Francisco_Jose_Freire'},
     {'name': 'Francesco Maria Appendini',
      'url': 'https://en.wikipedia.org/wiki/Francesco_Maria_Appendini'},
     {'name': 'Charles du Fresne, sieur du Cange',
      'url': 'https://en.wikipedia.org/wiki/Charles_du_Fresne,_sieur_du_Cange'},
     {'name': u'Charlotta Fr\xf6lich',
      'url': 'https://en.wikipedia.org/wiki/Charlotta_Fr%C3%B6lich'},
     {'name': 'Garcilaso de la Vega',
      'url': 'https://en.wikipedia.org/wiki/Inca_Garcilaso_de_la_Vega'},
     {'name': 'Erik Gustaf Geijer',
      'url': 'https://en.wikipedia.org/wiki/Erik_Gustaf_Geijer'},
     {'name': 'Edward Gibbon',
      'url': 'https://en.wikipedia.org/wiki/Edward_Gibbon'},
     {'name': 'George Grote', 'url': 'https://en.wikipedia.org/wiki/George_Grote'},
     {'name': 'Giambattista Vico',
      'url': 'https://en.wikipedia.org/wiki/Giambattista_Vico'},
     {'name': u'Fran\xe7ois Guizot',
      'url': 'https://en.wikipedia.org/wiki/Fran%C3%A7ois_Guizot'},
     {'name': 'Edward Hasted',
      'url': 'https://en.wikipedia.org/wiki/Edward_Hasted'},
     {'name': 'Sulayman al-Hawwat',
      'url': 'https://en.wikipedia.org/wiki/Sulayman_al-Hawwat'},
     {'name': 'Georg Wilhelm Friedrich Hegel',
      'url': 'https://en.wikipedia.org/wiki/Georg_Wilhelm_Friedrich_Hegel'},
     {'name': 'Alexander Hewat (or Hewatt)',
      'url': 'https://en.wikipedia.org/wiki/Alexander_Hewat'},
     {'name': 'Pieter Corneliszoon Hooft',
      'url': 'https://en.wikipedia.org/wiki/Pieter_Corneliszoon_Hooft'},
     {'name': 'Arild Huitfeldt',
      'url': 'https://en.wikipedia.org/wiki/Arild_Huitfeldt'},
     {'name': 'David Hume', 'url': 'https://en.wikipedia.org/wiki/David_Hume'},
     {'name': 'Thomas Hutchinson',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Hutchinson_(governor)'},
     {'name': 'Mohammed al-Ifrani',
      'url': 'https://en.wikipedia.org/wiki/Mohammed_al-Ifrani'},
     {'name': 'Nikolai Mikhailovich Karamzin',
      'url': 'https://en.wikipedia.org/wiki/Nikolai_Mikhailovich_Karamzin'},
     {'name': 'Geoffrey Keating',
      'url': 'https://en.wikipedia.org/wiki/Geoffrey_Keating'},
     {'name': 'Joachim Lelewel',
      'url': 'https://en.wikipedia.org/wiki/Joachim_Lelewel'},
     {'name': 'John Lingard', 'url': 'https://en.wikipedia.org/wiki/John_Lingard'},
     {'name': 'Anton Tomaz Linhart',
      'url': 'https://en.wikipedia.org/wiki/Anton_Tomaz_Linhart'},
     {'name': 'F.S.L. Lyons', 'url': 'https://en.wikipedia.org/wiki/F.S.L._Lyons'},
     {'name': 'Dubhaltach MacFhirbhisigh',
      'url': 'https://en.wikipedia.org/wiki/Dubhaltach_MacFhirbhisigh'},
     {'name': 'Jules Michelet',
      'url': 'https://en.wikipedia.org/wiki/Jules_Michelet'},
     {'name': u'Fran\xe7ois Mignet',
      'url': 'https://en.wikipedia.org/wiki/Fran%C3%A7ois_Mignet'},
     {'name': 'Christian Molbech',
      'url': 'https://en.wikipedia.org/wiki/Christian_Molbech'},
     {'name': 'Johann Lorenz Von Mosheim',
      'url': 'https://en.wikipedia.org/wiki/Johann_Lorenz_Von_Mosheim'},
     {'name': u'Johannes von M\xfcller',
      'url': 'https://en.wikipedia.org/wiki/Johannes_von_M%C3%BCller'},
     {'name': 'Ludovico Antonio Muratori',
      'url': 'https://en.wikipedia.org/wiki/Ludovico_Antonio_Muratori'},
     {'name': u'Louis-S\xe9bastien Le Nain de Tillemont',
      'url': 'https://en.wikipedia.org/wiki/Louis-S%C3%A9bastien_Le_Nain_de_Tillemont'},
     {'name': 'Barthold Georg Niebuhr',
      'url': 'https://en.wikipedia.org/wiki/Barthold_Georg_Niebuhr'},
     {'name': u'Tadhg \xd3g \xd3 Cian\xe1in',
      'url': 'https://en.wikipedia.org/wiki/Tadhg_%C3%93g_%C3%93_Cian%C3%A1in'},
     {'name': u'M\xedche\xe1l \xd3 Cl\xe9irigh',
      'url': 'https://en.wikipedia.org/wiki/M%C3%ADche%C3%A1l_%C3%93_Cl%C3%A9irigh'},
     {'name': u'Peregrine \xd3 Duibhgeannain',
      'url': 'https://en.wikipedia.org/wiki/Peregrine_%C3%93_Duibhgeannain'},
     {'name': u'C\xfa Choigcr\xedche \xd3 Cl\xe9irigh',
      'url': 'https://en.wikipedia.org/wiki/C%C3%BA_Choigcr%C3%ADche_%C3%93_Cl%C3%A9irigh'},
     {'name': u'Ruaidhr\xed \xd3 Flaithbheartaigh',
      'url': 'https://en.wikipedia.org/wiki/Ruaidhr%C3%AD_%C3%93_Flaithbheartaigh'},
     {'name': 'Zaharije Orfelin',
      'url': 'https://en.wikipedia.org/wiki/Zaharije_Orfelin'},
     {'name': 'Olaus Magnus', 'url': 'https://en.wikipedia.org/wiki/Olaus_Magnus'},
     {'name': u'Franti\u0161ek Palack\xfd',
      'url': 'https://en.wikipedia.org/wiki/Franti%C5%A1ek_Palack%C3%BD'},
     {'name': 'William H. Prescott',
      'url': 'https://en.wikipedia.org/wiki/William_H._Prescott'},
     {'name': 'Placido Puccinelli',
      'url': 'https://en.wikipedia.org/wiki/Placido_Puccinelli'},
     {'name': 'Mohammed al-Qadiri',
      'url': 'https://en.wikipedia.org/wiki/Mohammed_al-Qadiri'},
     {'name': 'Qian Daxin', 'url': 'https://en.wikipedia.org/wiki/Qian_Daxin'},
     {'name': 'Qian Qianyi', 'url': 'https://en.wikipedia.org/wiki/Qian_Qianyi'},
     {'name': 'David Ramsay',
      'url': 'https://en.wikipedia.org/wiki/David_Ramsay_(congressman)'},
     {'name': 'Leopold von Ranke',
      'url': 'https://en.wikipedia.org/wiki/Leopold_von_Ranke'},
     {'name': 'John F. Richards',
      'url': 'https://en.wikipedia.org/wiki/John_F._Richards'},
     {'name': 'Mikhail Shcherbatov',
      'url': 'https://en.wikipedia.org/wiki/Mikhail_Shcherbatov'},
     {'name': 'John Strype', 'url': 'https://en.wikipedia.org/wiki/John_Strype'},
     {'name': 'Vasily Tatishchev',
      'url': 'https://en.wikipedia.org/wiki/Vasily_Tatishchev'},
     {'name': 'Adolphe Thiers',
      'url': 'https://en.wikipedia.org/wiki/Adolphe_Thiers'},
     {'name': 'George Tucker',
      'url': 'https://en.wikipedia.org/wiki/George_Tucker_(politician)'},
     {'name': 'Voltaire', 'url': 'https://en.wikipedia.org/wiki/Voltaire'},
     {'name': 'Sir James Ware',
      'url': 'https://en.wikipedia.org/wiki/Sir_James_Ware'},
     {'name': 'Yu Deuk-gong', 'url': 'https://en.wikipedia.org/wiki/Yu_Deuk-gong'},
     {'name': 'Abu al-Qasim al-Zayyani',
      'url': 'https://en.wikipedia.org/wiki/Abu_al-Qasim_al-Zayyani'},
     {'name': 'Zhang Tingyu', 'url': 'https://en.wikipedia.org/wiki/Zhang_Tingyu'},
     {'name': 'Lord Acton',
      'url': 'https://en.wikipedia.org/wiki/John_Dalberg-Acton,_1st_Baron_Acton'},
     {'name': 'Henry Adams',
      'url': 'https://en.wikipedia.org/wiki/Henry_Brooks_Adams'},
     {'name': 'Grace Aguilar',
      'url': 'https://en.wikipedia.org/wiki/Grace_Aguilar'},
     {'name': 'Charles McLean Andrews',
      'url': 'https://en.wikipedia.org/wiki/Charles_McLean_Andrews'},
     {'name': 'Alfred von Arneth',
      'url': 'https://en.wikipedia.org/wiki/Alfred_von_Arneth'},
     {'name': 'Mikhail Artamonov',
      'url': 'https://en.wikipedia.org/wiki/Mikhail_Artamonov'},
     {'name': 'William Ashley',
      'url': 'https://en.wikipedia.org/wiki/William_Ashley_(economic_historian)'},
     {'name': 'Octave Aubry', 'url': 'https://en.wikipedia.org/wiki/Octave_Aubry'},
     {'name': u'Fran\xe7ois Victor Alphonse Aulard',
      'url': 'https://en.wikipedia.org/wiki/Fran%C3%A7ois_Victor_Alphonse_Aulard'},
     {'name': 'Zurab Avalishvili',
      'url': 'https://en.wikipedia.org/wiki/Zurab_Avalishvili'},
     {'name': 'Jacques Bainville',
      'url': 'https://en.wikipedia.org/wiki/Jacques_Bainville'},
     {'name': 'R. Mildred Barker',
      'url': 'https://en.wikipedia.org/wiki/R._Mildred_Barker'},
     {'name': 'Harry Elmer Barnes',
      'url': 'https://en.wikipedia.org/wiki/Harry_Elmer_Barnes'},
     {'name': 'Charles Bean', 'url': 'https://en.wikipedia.org/wiki/Charles_Bean'},
     {'name': 'Charles A. Beard',
      'url': 'https://en.wikipedia.org/wiki/Charles_A._Beard'},
     {'name': 'Mary Ritter Beard',
      'url': 'https://en.wikipedia.org/wiki/Mary_Ritter_Beard'},
     {'name': 'George Bancroft',
      'url': 'https://en.wikipedia.org/wiki/George_Bancroft'},
     {'name': 'Wilhelm Barthold',
      'url': 'https://en.wikipedia.org/wiki/Vasily_Bartold'},
     {'name': 'Winthrop Pickard Bell',
      'url': 'https://en.wikipedia.org/wiki/Winthrop_Pickard_Bell'},
     {'name': 'Hilaire Belloc',
      'url': 'https://en.wikipedia.org/wiki/Hilaire_Belloc'},
     {'name': 'Marc Bloch', 'url': 'https://en.wikipedia.org/wiki/Marc_Bloch'},
     {'name': 'Herbert Eugene Bolton',
      'url': 'https://en.wikipedia.org/wiki/Herbert_Eugene_Bolton'},
     {'name': 'George Williams Brown',
      'url': 'https://en.wikipedia.org/wiki/George_Williams_Brown'},
     {'name': 'Erich Brandenburg',
      'url': 'https://en.wikipedia.org/wiki/Erich_Brandenburg'},
     {'name': 'Otto Brunner', 'url': 'https://en.wikipedia.org/wiki/Otto_Brunner'},
     {'name': 'Geoffrey Bruun',
      'url': 'https://en.wikipedia.org/wiki/Geoffrey_Bruun'},
     {'name': 'Arthur Bryant',
      'url': 'https://en.wikipedia.org/wiki/Arthur_Bryant'},
     {'name': 'Henry Thomas Buckle',
      'url': 'https://en.wikipedia.org/wiki/Henry_Thomas_Buckle'},
     {'name': 'Jacob Burckhardt',
      'url': 'https://en.wikipedia.org/wiki/Jacob_Burckhardt'},
     {'name': 'John Hill Burton',
      'url': 'https://en.wikipedia.org/wiki/John_Hill_Burton'},
     {'name': 'J.B. Bury', 'url': 'https://en.wikipedia.org/wiki/J.B._Bury'},
     {'name': 'Helen Cam', 'url': 'https://en.wikipedia.org/wiki/Helen_Cam'},
     {'name': 'Pierre Caron',
      'url': 'https://en.wikipedia.org/wiki/Pierre_Caron_(historian)'},
     {'name': 'E.H. Carr', 'url': 'https://en.wikipedia.org/wiki/E.H._Carr'},
     {'name': u'Antonio C\xe1novas del Castillo',
      'url': 'https://en.wikipedia.org/wiki/Antonio_C%C3%A1novas_del_Castillo'},
     {'name': 'Henri Raymond Casgrain',
      'url': 'https://en.wikipedia.org/wiki/Henri_Raymond_Casgrain'},
     {'name': u'Am\xe9rico Castro',
      'url': 'https://en.wikipedia.org/wiki/Am%C3%A9rico_Castro'},
     {'name': 'Bruce Catton', 'url': 'https://en.wikipedia.org/wiki/Bruce_Catton'},
     {'name': 'Cesar de Bazancourt',
      'url': 'https://en.wikipedia.org/wiki/Baron_de_C%C3%A9sar_Bazancourt'},
     {'name': 'Nirad C. Chaudhuri',
      'url': 'https://en.wikipedia.org/wiki/Nirad_C._Chaudhuri'},
     {'name': 'Boris Chicherin',
      'url': 'https://en.wikipedia.org/wiki/Boris_Chicherin'},
     {'name': 'Hiram M. Chittenden',
      'url': 'https://en.wikipedia.org/wiki/Hiram_M._Chittenden'},
     {'name': 'Winston Churchill',
      'url': 'https://en.wikipedia.org/wiki/Winston_Churchill'},
     {'name': 'Augustin Cochin',
      'url': 'https://en.wikipedia.org/wiki/Augustin_Cochin_(historian)'},
     {'name': 'R. G. Collingwood',
      'url': 'https://en.wikipedia.org/wiki/R._G._Collingwood'},
     {'name': 'Julian Corbett',
      'url': 'https://en.wikipedia.org/wiki/Julian_Corbett'},
     {'name': u'Vladimir \u0106orovi\u0107',
      'url': 'https://en.wikipedia.org/wiki/Vladimir_%C4%86orovi%C4%87'},
     {'name': 'Avery Craven', 'url': 'https://en.wikipedia.org/wiki/Avery_Craven'},
     {'name': 'Edward Shepherd Creasy',
      'url': 'https://en.wikipedia.org/wiki/Edward_Shepherd_Creasy'},
     {'name': 'Margaret Campbell Speke Cruwys',
      'url': 'https://en.wikipedia.org/wiki/Margaret_Campbell_Speke_Cruwys'},
     {'name': 'Felix Dahn', 'url': 'https://en.wikipedia.org/wiki/Felix_Dahn'},
     {'name': 'Angie Debo', 'url': 'https://en.wikipedia.org/wiki/Angie_Debo'},
     {'name': u'L\xe9opold Delisle',
      'url': 'https://en.wikipedia.org/wiki/L%C3%A9opold_Victor_Delisle'},
     {'name': 'Bernard DeVoto',
      'url': 'https://en.wikipedia.org/wiki/Bernard_DeVoto'},
     {'name': 'William Dodd',
      'url': 'https://en.wikipedia.org/wiki/William_Dodd_(ambassador)'},
     {'name': 'David C. Douglas',
      'url': 'https://en.wikipedia.org/wiki/David_C._Douglas'},
     {'name': 'Johann Gustav Droysen',
      'url': 'https://en.wikipedia.org/wiki/Johann_Gustav_Droysen'},
     {'name': 'Ariel Durant', 'url': 'https://en.wikipedia.org/wiki/Ariel_Durant'},
     {'name': 'Will Durant', 'url': 'https://en.wikipedia.org/wiki/Will_Durant'},
     {'name': 'Mary Anne Everett Green',
      'url': 'https://en.wikipedia.org/wiki/Mary_Anne_Everett_Green'},
     {'name': 'Ephraim Emerton',
      'url': 'https://en.wikipedia.org/wiki/Ephraim_Emerton'},
     {'name': 'Cyril Falls', 'url': 'https://en.wikipedia.org/wiki/Cyril_Falls'},
     {'name': 'Keith Feiling',
      'url': 'https://en.wikipedia.org/wiki/Keith_Feiling'},
     {'name': 'Herbert Feis', 'url': 'https://en.wikipedia.org/wiki/Herbert_Feis'},
     {'name': 'Lucien Febvre',
      'url': 'https://en.wikipedia.org/wiki/Lucien_Febvre'},
     {'name': 'Charles Harding Firth',
      'url': 'https://en.wikipedia.org/wiki/Charles_Harding_Firth'},
     {'name': 'Walter Lynwood Fleming',
      'url': 'https://en.wikipedia.org/wiki/Walter_Lynwood_Fleming'},
     {'name': 'Edward Augustus Freeman',
      'url': 'https://en.wikipedia.org/wiki/Edward_Augustus_Freeman'},
     {'name': 'James Anthony Froude',
      'url': 'https://en.wikipedia.org/wiki/James_Anthony_Froude'},
     {'name': 'J.F.C. Fuller',
      'url': 'https://en.wikipedia.org/wiki/J.F.C._Fuller'},
     {'name': 'Frantz Funck-Brentano',
      'url': 'https://en.wikipedia.org/wiki/Frantz_Funck-Brentano'},
     {'name': 'John Sydenham Furnivall',
      'url': 'https://en.wikipedia.org/wiki/John_Sydenham_Furnivall'},
     {'name': 'Numa Denis Fustel de Coulanges',
      'url': 'https://en.wikipedia.org/wiki/Numa_Denis_Fustel_de_Coulanges'},
     {'name': u'Fran\xe7ois-Louis Ganshof',
      'url': 'https://en.wikipedia.org/wiki/Fran%C3%A7ois-Louis_Ganshof'},
     {'name': 'Samuel Rawson Gardiner',
      'url': 'https://en.wikipedia.org/wiki/Samuel_Rawson_Gardiner'},
     {'name': 'Pieter Geyl', 'url': 'https://en.wikipedia.org/wiki/Pieter_Geyl'},
     {'name': 'Lawrence Henry Gipson',
      'url': 'https://en.wikipedia.org/wiki/Lawrence_Henry_Gipson'},
     {'name': 'Arthur Giry', 'url': 'https://en.wikipedia.org/wiki/Arthur_Giry'},
     {'name': 'Gustave Glotz',
      'url': 'https://en.wikipedia.org/wiki/Gustave_Glotz'},
     {'name': 'George Peabody Gooch',
      'url': 'https://en.wikipedia.org/wiki/George_Peabody_Gooch'},
     {'name': 'Timofey Granovsky',
      'url': 'https://en.wikipedia.org/wiki/Timofey_Granovsky'},
     {'name': 'John Richard Green',
      'url': 'https://en.wikipedia.org/wiki/John_Richard_Green'},
     {'name': 'Lionel Groulx',
      'url': 'https://en.wikipedia.org/wiki/Lionel_Groulx'},
     {'name': u'Ren\xe9 Grousset',
      'url': 'https://en.wikipedia.org/wiki/Ren%C3%A9_Grousset'},
     {'name': 'Louis Halphen',
      'url': 'https://en.wikipedia.org/wiki/Louis_Halphen'},
     {'name': 'Clarence H. Haring',
      'url': 'https://en.wikipedia.org/wiki/Clarence_H._Haring'},
     {'name': 'Charles H. Haskins',
      'url': 'https://en.wikipedia.org/wiki/Charles_H._Haskins'},
     {'name': 'Henri Hauser', 'url': 'https://en.wikipedia.org/wiki/Henri_Hauser'},
     {'name': 'Julien Havet', 'url': 'https://en.wikipedia.org/wiki/Julien_Havet'},
     {'name': 'Paul Hazard', 'url': 'https://en.wikipedia.org/wiki/Paul_Hazard'},
     {'name': 'Eli Heckscher',
      'url': 'https://en.wikipedia.org/wiki/Eli_Heckscher'},
     {'name': 'Auguste Himly',
      'url': 'https://en.wikipedia.org/wiki/Auguste_Himly'},
     {'name': u'Mih\xe1ly Horv\xe1th',
      'url': 'https://en.wikipedia.org/wiki/Mih%C3%A1ly_Horv%C3%A1th'},
     {'name': 'Johan Huizinga',
      'url': 'https://en.wikipedia.org/wiki/Johan_Huizinga'},
     {'name': 'Ibn Zaydan', 'url': 'https://en.wikipedia.org/wiki/Ibn_Zaydan'},
     {'name': 'Dmitry Ilovaisky',
      'url': 'https://en.wikipedia.org/wiki/Dmitry_Ilovaisky'},
     {'name': 'Harold Innis', 'url': 'https://en.wikipedia.org/wiki/Harold_Innis'},
     {'name': 'Mohammed ibn Jaafar al-Kattani',
      'url': 'https://en.wikipedia.org/wiki/Mohammed_ibn_Jaafar_al-Kattani'},
     {'name': 'Muhammad Jaber',
      'url': 'https://en.wikipedia.org/wiki/Muhammad_Jaber'},
     {'name': 'William James (naval historian)',
      'url': 'https://en.wikipedia.org/wiki/William_James_(naval_historian)'},
     {'name': 'Ivane Javakhishvili',
      'url': 'https://en.wikipedia.org/wiki/Ivane_Javakhishvili'},
     {'name': 'Samuel Kamakau',
      'url': 'https://en.wikipedia.org/wiki/Samuel_Kamakau'},
     {'name': 'Konstantin Kavelin',
      'url': 'https://en.wikipedia.org/wiki/Konstantin_Kavelin'},
     {'name': 'Hans Kelsen', 'url': 'https://en.wikipedia.org/wiki/Hans_Kelsen'},
     {'name': 'Philip Moore Callow Kermode',
      'url': 'https://en.wikipedia.org/wiki/P._M._C._Kermode'},
     {'name': 'Alexander William Kinglake',
      'url': 'https://en.wikipedia.org/wiki/Alexander_William_Kinglake'},
     {'name': 'William Kingsford',
      'url': 'https://en.wikipedia.org/wiki/William_Kingsford'},
     {'name': 'Vasily Klyuchevsky',
      'url': 'https://en.wikipedia.org/wiki/Vasily_Klyuchevsky'},
     {'name': 'David Knowles',
      'url': 'https://en.wikipedia.org/wiki/David_Knowles_(scholar)'},
     {'name': 'Dudley Wright Knox',
      'url': 'https://en.wikipedia.org/wiki/Dudley_Wright_Knox'},
     {'name': u'Ludwig von K\xf6chel',
      'url': 'https://en.wikipedia.org/wiki/Ludwig_von_K%C3%B6chel'},
     {'name': u'Mihail Kog\u0103lniceanu',
      'url': 'https://en.wikipedia.org/wiki/Mihail_Kog%C4%83lniceanu'},
     {'name': 'Hans Kohn', 'url': 'https://en.wikipedia.org/wiki/Hans_Kohn'},
     {'name': 'Nikodim Kondakov',
      'url': 'https://en.wikipedia.org/wiki/Nikodim_Kondakov'},
     {'name': 'Nikolay Kostomarov',
      'url': 'https://en.wikipedia.org/wiki/Nikolay_Kostomarov'},
     {'name': 'Godefroid Kurth',
      'url': 'https://en.wikipedia.org/wiki/Godefroid_Kurth'},
     {'name': 'Leonard Woods Labaree',
      'url': 'https://en.wikipedia.org/wiki/Leonard_Woods_Labaree'},
     {'name': 'William L. Langer',
      'url': 'https://en.wikipedia.org/wiki/William_L._Langer'},
     {'name': 'John Knox Laughton',
      'url': 'https://en.wikipedia.org/wiki/John_Knox_Laughton'},
     {'name': 'Ernest Lavisse',
      'url': 'https://en.wikipedia.org/wiki/Ernest_Lavisse'},
     {'name': 'Georges Lefebvre',
      'url': 'https://en.wikipedia.org/wiki/Georges_Lefebvre'},
     {'name': 'Liang Qichao', 'url': 'https://en.wikipedia.org/wiki/Liang_Qichao'},
     {'name': 'B.H. Liddell Hart',
      'url': 'https://en.wikipedia.org/wiki/B.H._Liddell_Hart'},
     {'name': 'John Edward Lloyd',
      'url': 'https://en.wikipedia.org/wiki/John_Edward_Lloyd'},
     {'name': 'Ferdinand Lot',
      'url': 'https://en.wikipedia.org/wiki/Ferdinand_Lot'},
     {'name': 'Arthur R.M. Lower',
      'url': 'https://en.wikipedia.org/wiki/Arthur_R.M._Lower'},
     {'name': 'Thomas Macaulay',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Macaulay'},
     {'name': 'William Archibald Mackintosh',
      'url': 'https://en.wikipedia.org/wiki/William_Archibald_Mackintosh'},
     {'name': 'J. D. Mackie', 'url': 'https://en.wikipedia.org/wiki/J._D._Mackie'},
     {'name': 'Frederic William Maitland',
      'url': 'https://en.wikipedia.org/wiki/Frederic_William_Maitland'},
     {'name': 'Alfred Thayer Mahan',
      'url': 'https://en.wikipedia.org/wiki/Alfred_Thayer_Mahan'},
     {'name': 'Ramesh Chandra Majumdar',
      'url': 'https://en.wikipedia.org/wiki/Ramesh_Chandra_Majumdar'},
     {'name': 'J.A.R. Marriott',
      'url': 'https://en.wikipedia.org/wiki/John_Marriott_(British_politician)'},
     {'name': 'Albert Mathiez',
      'url': 'https://en.wikipedia.org/wiki/Albert_Mathiez'},
     {'name': 'Karl Marx', 'url': 'https://en.wikipedia.org/wiki/Karl_Marx'},
     {'name': 'Friedrich Meinecke',
      'url': 'https://en.wikipedia.org/wiki/Friedrich_Meinecke'},
     {'name': 'Krste Misirkov',
      'url': 'https://en.wikipedia.org/wiki/Krste_Misirkov'},
     {'name': 'Auguste Molinier',
      'url': 'https://en.wikipedia.org/wiki/Auguste_Molinier'},
     {'name': 'Theodor Mommsen',
      'url': 'https://en.wikipedia.org/wiki/Theodor_Mommsen'},
     {'name': 'Indro Montanelli',
      'url': 'https://en.wikipedia.org/wiki/Indro_Montanelli'},
     {'name': 'Alfred Morel-Fatio',
      'url': 'https://en.wikipedia.org/wiki/Alfred_Morel-Fatio'},
     {'name': 'Samuel Eliot Morison',
      'url': 'https://en.wikipedia.org/wiki/Samuel_Eliot_Morison'},
     {'name': 'Lewis Mumford',
      'url': 'https://en.wikipedia.org/wiki/Lewis_Mumford'},
     {'name': 'Lewis Bernstein Namier',
      'url': 'https://en.wikipedia.org/wiki/Lewis_Bernstein_Namier'},
     {'name': 'Ahmad ibn Khalid al-Nasiri',
      'url': 'https://en.wikipedia.org/wiki/Ahmad_ibn_Khalid_al-Nasiri'},
     {'name': 'J. E. Neale', 'url': 'https://en.wikipedia.org/wiki/J._E._Neale'},
     {'name': 'Allan Nevins', 'url': 'https://en.wikipedia.org/wiki/Allan_Nevins'},
     {'name': 'A. P. Newton', 'url': 'https://en.wikipedia.org/wiki/A._P._Newton'},
     {'name': u'Stojan Novakovi\u0107',
      'url': 'https://en.wikipedia.org/wiki/Stojan_Novakovi%C4%87'},
     {'name': 'Charles Oman', 'url': 'https://en.wikipedia.org/wiki/Charles_Oman'},
     {'name': 'Herbert L. Osgood',
      'url': 'https://en.wikipedia.org/wiki/Herbert_L._Osgood'},
     {'name': 'Cesare Paoli', 'url': 'https://en.wikipedia.org/wiki/Cesare_Paoli'},
     {'name': 'Gaston Paris', 'url': 'https://en.wikipedia.org/wiki/Gaston_Paris'},
     {'name': 'Herbert Paul', 'url': 'https://en.wikipedia.org/wiki/Herbert_Paul'},
     {'name': 'Henry Francis Pelham',
      'url': 'https://en.wikipedia.org/wiki/Henry_Francis_Pelham'},
     {'name': 'Samuel W. Pennypacker',
      'url': 'https://en.wikipedia.org/wiki/Samuel_W._Pennypacker'},
     {'name': 'Dexter Perkins',
      'url': 'https://en.wikipedia.org/wiki/Dexter_Perkins'},
     {'name': 'Henri Pirenne',
      'url': 'https://en.wikipedia.org/wiki/Henri_Pirenne'},
     {'name': 'Sergey Platonov',
      'url': 'https://en.wikipedia.org/wiki/Sergey_Platonov'},
     {'name': 'Ivy Pinchbeck',
      'url': 'https://en.wikipedia.org/wiki/Ivy_Pinchbeck'},
     {'name': 'Eileen Power', 'url': 'https://en.wikipedia.org/wiki/Eileen_Power'},
     {'name': 'F. M. Powicke',
      'url': 'https://en.wikipedia.org/wiki/F._M._Powicke'},
     {'name': 'H. F. M. Prescott',
      'url': 'https://en.wikipedia.org/wiki/H._F._M._Prescott'},
     {'name': 'Datto Vaman Potdar',
      'url': 'https://en.wikipedia.org/wiki/Datto_Vaman_Potdar'},
     {'name': 'Jules Quicherat',
      'url': 'https://en.wikipedia.org/wiki/Jules_Etienne_Joseph_Quicherat'},
     {'name': 'William Pember Reeves',
      'url': 'https://en.wikipedia.org/wiki/William_Pember_Reeves'},
     {'name': 'Pierre Renouvin',
      'url': 'https://en.wikipedia.org/wiki/Pierre_Renouvin'},
     {'name': 'James Riker', 'url': 'https://en.wikipedia.org/wiki/James_Riker'},
     {'name': 'B. H. Roberts',
      'url': 'https://en.wikipedia.org/wiki/B._H._Roberts'},
     {'name': 'James Harvey Robinson',
      'url': 'https://en.wikipedia.org/wiki/James_Harvey_Robinson'},
     {'name': 'Theodore Roosevelt',
      'url': 'https://en.wikipedia.org/wiki/Theodore_Roosevelt'},
     {'name': 'Simon Rutar', 'url': 'https://en.wikipedia.org/wiki/Simon_Rutar'},
     {'name': 'Ilarion Ruvarac',
      'url': 'https://en.wikipedia.org/wiki/Ilarion_Ruvarac'},
     {'name': 'Abram L. Sachar',
      'url': 'https://en.wikipedia.org/wiki/Abram_L._Sachar'},
     {'name': 'George Sarton',
      'url': 'https://en.wikipedia.org/wiki/George_Sarton'},
     {'name': 'Gustave Schlumberger',
      'url': 'https://en.wikipedia.org/wiki/Gustave_Schlumberger'},
     {'name': 'John Robert Seeley',
      'url': 'https://en.wikipedia.org/wiki/John_Robert_Seeley'},
     {'name': 'Sergey Solovyov',
      'url': 'https://en.wikipedia.org/wiki/Sergey_Solovyov'},
     {'name': 'Govind Sakharam Sardesai',
      'url': 'https://en.wikipedia.org/wiki/Govind_Sakharam_Sardesai'},
     {'name': 'Adam Shortt', 'url': 'https://en.wikipedia.org/wiki/Adam_Shortt'},
     {'name': 'Goldwin Smith',
      'url': 'https://en.wikipedia.org/wiki/Goldwin_Smith'},
     {'name': 'Oswald Spengler',
      'url': 'https://en.wikipedia.org/wiki/Oswald_Spengler'},
     {'name': 'Shin Chaeho', 'url': 'https://en.wikipedia.org/wiki/Shin_Chaeho'},
     {'name': 'Frank Stenton',
      'url': 'https://en.wikipedia.org/wiki/Frank_Stenton'},
     {'name': 'Doris Mary Stenton',
      'url': 'https://en.wikipedia.org/wiki/Doris_Mary_Stenton'},
     {'name': 'William Stubbs',
      'url': 'https://en.wikipedia.org/wiki/William_Stubbs'},
     {'name': 'Hippolyte Taine',
      'url': 'https://en.wikipedia.org/wiki/Hippolyte_Taine'},
     {'name': 'Frank Bigelow Tarbell',
      'url': 'https://en.wikipedia.org/wiki/Frank_Bigelow_Tarbell'},
     {'name': 'Yevgeny Tarle',
      'url': 'https://en.wikipedia.org/wiki/Yevgeny_Tarle'},
     {'name': 'A. Wyatt Tilby',
      'url': 'https://en.wikipedia.org/wiki/A._Wyatt_Tilby'},
     {'name': 'Alexis de Tocqueville',
      'url': 'https://en.wikipedia.org/wiki/Alexis_de_Tocqueville'},
     {'name': 'Zacharias Topelius',
      'url': 'https://en.wikipedia.org/wiki/Zacharias_Topelius'},
     {'name': 'Thomas Frederick Tout',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Frederick_Tout'},
     {'name': 'Arnold J. Toynbee',
      'url': 'https://en.wikipedia.org/wiki/Arnold_J._Toynbee'},
     {'name': 'Heinrich Gotthard von Treitschke',
      'url': 'https://en.wikipedia.org/wiki/Heinrich_Gotthard_von_Treitschke'},
     {'name': 'George Macaulay Trevelyan',
      'url': 'https://en.wikipedia.org/wiki/George_Macaulay_Trevelyan'},
     {'name': 'Mikheil Tsereteli',
      'url': 'https://en.wikipedia.org/wiki/Mikheil_Tsereteli'},
     {'name': 'Frank Underhill',
      'url': 'https://en.wikipedia.org/wiki/Frank_Underhill'},
     {'name': 'Paul Vinogradoff',
      'url': 'https://en.wikipedia.org/wiki/Paul_Vinogradoff'},
     {'name': 'Spencer Walpole',
      'url': 'https://en.wikipedia.org/wiki/Spencer_Walpole'},
     {'name': 'Curt Weibull', 'url': 'https://en.wikipedia.org/wiki/Curt_Weibull'},
     {'name': 'Lauritz Weibull',
      'url': 'https://en.wikipedia.org/wiki/Lauritz_Weibull'},
     {'name': 'Charles Webster',
      'url': 'https://en.wikipedia.org/wiki/Charles_Webster_(historian)'},
     {'name': 'Mary Wilhelmine Williams',
      'url': 'https://en.wikipedia.org/wiki/Mary_Wilhelmine_Williams'},
     {'name': 'Spenser Wilkinson',
      'url': 'https://en.wikipedia.org/wiki/Spenser_Wilkinson'},
     {'name': 'James A. Williamson',
      'url': 'https://en.wikipedia.org/wiki/James_Williamson_(historian)'},
     {'name': 'Justin Winsor',
      'url': 'https://en.wikipedia.org/wiki/Justin_Winsor'},
     {'name': 'Llewellyn Woodward',
      'url': 'https://en.wikipedia.org/wiki/Llewellyn_Woodward'},
     {'name': 'George MacKinnon Wrong',
      'url': 'https://en.wikipedia.org/wiki/George_MacKinnon_Wrong'},
     {'name': 'Yi Byeongdo', 'url': 'https://en.wikipedia.org/wiki/Yi_Byeongdo'},
     {'name': 'Faddei Zielinski',
      'url': 'https://en.wikipedia.org/wiki/Faddei_Zielinski'},
     {'name': 'Raouf Abbas', 'url': 'https://en.wikipedia.org/wiki/Raouf_Abbas'},
     {'name': 'Irving Abella',
      'url': 'https://en.wikipedia.org/wiki/Irving_Abella'},
     {'name': 'Aberjhani', 'url': 'https://en.wikipedia.org/wiki/Aberjhani'},
     {'name': 'David Abulafia',
      'url': 'https://en.wikipedia.org/wiki/David_Abulafia'},
     {'name': 'Ezequiel Adamovsky',
      'url': 'https://en.wikipedia.org/wiki/Ezequiel_Adamovsky'},
     {'name': 'Donald Adamson',
      'url': 'https://en.wikipedia.org/wiki/Donald_Adamson'},
     {'name': 'Teodoro Agoncillo',
      'url': 'https://en.wikipedia.org/wiki/Teodoro_Agoncillo'},
     {'name': 'Robert G. Albion',
      'url': 'https://en.wikipedia.org/wiki/Robert_G._Albion'},
     {'name': 'Dean C. Allard',
      'url': 'https://en.wikipedia.org/wiki/Dean_C._Allard'},
     {'name': 'Robert C. Allen',
      'url': 'https://en.wikipedia.org/wiki/Robert_C._Allen'},
     {'name': 'Gar Alperovitz',
      'url': 'https://en.wikipedia.org/wiki/Gar_Alperovitz'},
     {'name': 'Ida Altman', 'url': 'https://en.wikipedia.org/wiki/Ida_Altman'},
     {'name': 'Abbas Amanat', 'url': 'https://en.wikipedia.org/wiki/Abbas_Amanat'},
     {'name': 'Mor Altshuler',
      'url': 'https://en.wikipedia.org/wiki/Mor_Altshuler'},
     {'name': 'Henri Amouroux',
      'url': 'https://en.wikipedia.org/wiki/Henri_Amouroux'},
     {'name': 'Stephen Ambrose',
      'url': 'https://en.wikipedia.org/wiki/Stephen_Ambrose'},
     {'name': 'Perry Anderson',
      'url': 'https://en.wikipedia.org/wiki/Perry_Anderson'},
     {'name': 'Joyce Appleby',
      'url': 'https://en.wikipedia.org/wiki/Joyce_Appleby'},
     {'name': 'Herbert Aptheker',
      'url': 'https://en.wikipedia.org/wiki/Herbert_Aptheker'},
     {'name': 'Leonie Archer',
      'url': 'https://en.wikipedia.org/wiki/Leonie_Archer'},
     {'name': u'Philippe Ari\xe8s',
      'url': 'https://en.wikipedia.org/wiki/Philippe_Ari%C3%A8s'},
     {'name': 'Karen Armstrong',
      'url': 'https://en.wikipedia.org/wiki/Karen_Armstrong'},
     {'name': 'Leonard J. Arrington',
      'url': 'https://en.wikipedia.org/wiki/Leonard_J._Arrington'},
     {'name': 'Thomas Asbridge',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Asbridge'},
     {'name': 'Maurice Ashley',
      'url': 'https://en.wikipedia.org/wiki/Maurice_Ashley_(historian)'},
     {'name': 'Paul Avrich', 'url': 'https://en.wikipedia.org/wiki/Paul_Avrich'},
     {'name': 'Ali Azaykou', 'url': 'https://en.wikipedia.org/wiki/Ali_Azaykou'},
     {'name': 'Eiichiro Azuma',
      'url': 'https://en.wikipedia.org/wiki/Eiichiro_Azuma'},
     {'name': 'Nigel Bagnall',
      'url': 'https://en.wikipedia.org/wiki/Nigel_Bagnall'},
     {'name': 'Bernard Bailyn',
      'url': 'https://en.wikipedia.org/wiki/Bernard_Bailyn'},
     {'name': 'David E. Barclay',
      'url': 'https://en.wikipedia.org/wiki/David_E._Barclay'},
     {'name': 'Juliet Barker',
      'url': 'https://en.wikipedia.org/wiki/Juliet_Barker'},
     {'name': 'Frank Barlow',
      'url': 'https://en.wikipedia.org/wiki/Frank_Barlow_(historian)'},
     {'name': 'Linda Diane Barnes',
      'url': 'https://en.wikipedia.org/wiki/Linda_Diane_Barnes'},
     {'name': 'G.W.S. Barrow',
      'url': 'https://en.wikipedia.org/wiki/G.W.S._Barrow'},
     {'name': 'H. Arnold Barton',
      'url': 'https://en.wikipedia.org/wiki/H._Arnold_Barton'},
     {'name': 'Paul R. Bartrop',
      'url': 'https://en.wikipedia.org/wiki/Paul_R._Bartrop'},
     {'name': 'Jacques Barzun',
      'url': 'https://en.wikipedia.org/wiki/Jacques_Barzun'},
     {'name': 'Jorge Basadre',
      'url': 'https://en.wikipedia.org/wiki/Jorge_Basadre'},
     {'name': 'Hanna Batatu', 'url': 'https://en.wikipedia.org/wiki/Hanna_Batatu'},
     {'name': 'K. Jack Bauer',
      'url': 'https://en.wikipedia.org/wiki/K._Jack_Bauer'},
     {'name': 'Yehuda Bauer', 'url': 'https://en.wikipedia.org/wiki/Yehuda_Bauer'},
     {'name': 'Stephen B. Baxter',
      'url': 'https://en.wikipedia.org/wiki/Stephen_B._Baxter'},
     {'name': 'David Bebbington',
      'url': 'https://en.wikipedia.org/wiki/David_Bebbington'},
     {'name': 'Antony Beevor',
      'url': 'https://en.wikipedia.org/wiki/Antony_Beevor'},
     {'name': 'James Belich',
      'url': 'https://en.wikipedia.org/wiki/James_Belich_(historian)'},
     {'name': 'Abdelmajid Benjelloun',
      'url': 'https://en.wikipedia.org/wiki/Abdelmajid_Benjelloun_(historian)'},
     {'name': 'Laurence Bergreen',
      'url': 'https://en.wikipedia.org/wiki/Laurence_Bergreen'},
     {'name': 'Isaiah Berlin',
      'url': 'https://en.wikipedia.org/wiki/Isaiah_Berlin'},
     {'name': 'Michael Beschloss',
      'url': 'https://en.wikipedia.org/wiki/Michael_Beschloss'},
     {'name': 'Nicholas Bethell',
      'url': 'https://en.wikipedia.org/wiki/Nicholas_Bethell'},
     {'name': 'Anthony Birley',
      'url': 'https://en.wikipedia.org/wiki/Anthony_Birley'},
     {'name': 'David Blackbourn',
      'url': 'https://en.wikipedia.org/wiki/David_Blackbourn'},
     {'name': 'Geoffrey Blainey',
      'url': 'https://en.wikipedia.org/wiki/Geoffrey_Blainey'},
     {'name': 'Gisela Bock', 'url': 'https://en.wikipedia.org/wiki/Gisela_Bock'},
     {'name': 'Brian Bond', 'url': 'https://en.wikipedia.org/wiki/Brian_Bond'},
     {'name': 'Daniel J. Boorstin',
      'url': 'https://en.wikipedia.org/wiki/Daniel_J._Boorstin'},
     {'name': 'Georges Bordonove',
      'url': 'https://en.wikipedia.org/wiki/Georges_Bordonove'},
     {'name': 'John Boswell', 'url': 'https://en.wikipedia.org/wiki/John_Boswell'},
     {'name': 'Robert Bothwell',
      'url': 'https://en.wikipedia.org/wiki/Robert_Bothwell'},
     {'name': u'G\xe9rard Bouchard',
      'url': 'https://en.wikipedia.org/wiki/G%C3%A9rard_Bouchard'},
     {'name': 'Joanna Bourke',
      'url': 'https://en.wikipedia.org/wiki/Joanna_Bourke'},
     {'name': 'Paul S. Boyer',
      'url': 'https://en.wikipedia.org/wiki/Paul_S._Boyer'},
     {'name': 'Karl Dietrich Bracher',
      'url': 'https://en.wikipedia.org/wiki/Karl_Dietrich_Bracher'},
     {'name': 'Jim Bradbury', 'url': 'https://en.wikipedia.org/wiki/Jim_Bradbury'},
     {'name': 'James C. Bradford',
      'url': 'https://en.wikipedia.org/wiki/James_C._Bradford'},
     {'name': 'David Brading',
      'url': 'https://en.wikipedia.org/wiki/David_Brading'},
     {'name': 'William Brandon',
      'url': 'https://en.wikipedia.org/wiki/William_Brandon_(author)'},
     {'name': 'Fernand Braudel',
      'url': 'https://en.wikipedia.org/wiki/Fernand_Braudel'},
     {'name': 'Ahron Bregman',
      'url': 'https://en.wikipedia.org/wiki/Ahron_Bregman'},
     {'name': 'Carl Bridenbaugh',
      'url': 'https://en.wikipedia.org/wiki/Carl_Bridenbaugh'},
     {'name': 'Timothy Brook',
      'url': 'https://en.wikipedia.org/wiki/Timothy_Brook_(historian)'},
     {'name': 'Martin Broszat',
      'url': 'https://en.wikipedia.org/wiki/Martin_Broszat'},
     {'name': 'Peter Brown',
      'url': 'https://en.wikipedia.org/wiki/Peter_Brown_(historian)'},
     {'name': 'Christopher Browning',
      'url': 'https://en.wikipedia.org/wiki/Christopher_Browning'},
     {'name': 'Alan Bullock', 'url': 'https://en.wikipedia.org/wiki/Alan_Bullock'},
     {'name': 'Peter Burke',
      'url': 'https://en.wikipedia.org/wiki/Peter_Burke_(historian)'},
     {'name': 'Briton C. Busch',
      'url': 'https://en.wikipedia.org/wiki/Briton_C._Busch'},
     {'name': 'Richard Bushman',
      'url': 'https://en.wikipedia.org/wiki/Richard_Bushman'},
     {'name': 'Herbert Butterfield',
      'url': 'https://en.wikipedia.org/wiki/Herbert_Butterfield'},
     {'name': 'Angus Calder', 'url': 'https://en.wikipedia.org/wiki/Angus_Calder'},
     {'name': 'Julio Caro Baroja',
      'url': 'https://en.wikipedia.org/wiki/Julio_Caro_Baroja'},
     {'name': 'Sir Raymond Carr',
      'url': 'https://en.wikipedia.org/wiki/Raymond_Carr'},
     {'name': 'Paul Cartledge',
      'url': 'https://en.wikipedia.org/wiki/Paul_Cartledge'},
     {'name': 'Lionel Casson',
      'url': 'https://en.wikipedia.org/wiki/Lionel_Casson'},
     {'name': 'Boris Celovsky',
      'url': 'https://en.wikipedia.org/wiki/Borivoj_Celovsky'},
     {'name': 'Howard I. Chapelle',
      'url': 'https://en.wikipedia.org/wiki/Howard_I._Chapelle'},
     {'name': 'Maher Charif', 'url': 'https://en.wikipedia.org/wiki/Maher_Charif'},
     {'name': 'Iris Chang', 'url': 'https://en.wikipedia.org/wiki/Iris_Chang'},
     {'name': 'Louis Chevalier',
      'url': 'https://en.wikipedia.org/wiki/Louis_Chevalier'},
     {'name': 'Thomas Childers',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Childers'},
     {'name': 'I. R. Christie',
      'url': 'https://en.wikipedia.org/wiki/I._R._Christie'},
     {'name': 'Alexander Campbell Cheyne',
      'url': 'https://en.wikipedia.org/wiki/Alexander_Campbell_Cheyne'},
     {'name': 'Satyabrata Rai Chowdhuri',
      'url': 'https://en.wikipedia.org/wiki/Satyabrata_Rai_Chowdhuri'},
     {'name': 'Alan Clark', 'url': 'https://en.wikipedia.org/wiki/Alan_Clark'},
     {'name': 'Christopher Clark',
      'url': 'https://en.wikipedia.org/wiki/Chris_Clark_(historian)'},
     {'name': 'J.C.D. Clark', 'url': 'https://en.wikipedia.org/wiki/J.C.D._Clark'},
     {'name': 'Manning Clark',
      'url': 'https://en.wikipedia.org/wiki/Manning_Clark'},
     {'name': 'Patrick Collinson',
      'url': 'https://en.wikipedia.org/wiki/Patrick_Collinson'},
     {'name': 'Robert Conquest',
      'url': 'https://en.wikipedia.org/wiki/Robert_Conquest'},
     {'name': 'Margaret Conrad',
      'url': 'https://en.wikipedia.org/wiki/Margaret_Conrad'},
     {'name': u'Vladimir \u0106orovi\u0107',
      'url': 'https://en.wikipedia.org/wiki/Vladimir_%C4%86orovi%C4%87'},
     {'name': 'Peter Cottrell',
      'url': 'https://en.wikipedia.org/wiki/Peter_Cottrell'},
     {'name': 'Gordon A. Craig',
      'url': 'https://en.wikipedia.org/wiki/Gordon_A._Craig'},
     {'name': 'Donald Creighton',
      'url': 'https://en.wikipedia.org/wiki/Donald_Creighton'},
     {'name': 'Vincent Cronin',
      'url': 'https://en.wikipedia.org/wiki/Vincent_Cronin'},
     {'name': 'William Cronon',
      'url': 'https://en.wikipedia.org/wiki/William_Cronon'},
     {'name': 'Pamela Kyle Crossley',
      'url': 'https://en.wikipedia.org/wiki/Pamela_Kyle_Crossley'},
     {'name': 'Dan Cruickshank',
      'url': 'https://en.wikipedia.org/wiki/Dan_Cruickshank'},
     {'name': 'Barry Cunliffe',
      'url': 'https://en.wikipedia.org/wiki/Barry_Cunliffe'},
     {'name': 'John Shelton Curtiss',
      'url': 'https://en.wikipedia.org/wiki/John_Shelton_Curtiss'},
     {'name': 'Robert Dallek',
      'url': 'https://en.wikipedia.org/wiki/Robert_Dallek'},
     {'name': 'Vahakn N. Dadrian',
      'url': 'https://en.wikipedia.org/wiki/Vahakn_N._Dadrian'},
     {'name': 'David B. Danbom',
      'url': 'https://en.wikipedia.org/wiki/David_B._Danbom'},
     {'name': 'Ahmad Hasan Dani',
      'url': 'https://en.wikipedia.org/wiki/Ahmad_Hasan_Dani'},
     {'name': 'Robert Darnton',
      'url': 'https://en.wikipedia.org/wiki/Robert_Darnton'},
     {'name': 'Lucy Dawidowicz',
      'url': 'https://en.wikipedia.org/wiki/Lucy_Dawidowicz'},
     {'name': 'Saul David', 'url': 'https://en.wikipedia.org/wiki/Saul_David'},
     {'name': 'John Davies',
      'url': 'https://en.wikipedia.org/wiki/John_Davies_(historian)'},
     {'name': 'Norman Davies',
      'url': 'https://en.wikipedia.org/wiki/Norman_Davies'},
     {'name': 'Natalie Zemon Davis',
      'url': 'https://en.wikipedia.org/wiki/Natalie_Zemon_Davis'},
     {'name': 'Kenneth S. Davis',
      'url': 'https://en.wikipedia.org/wiki/Kenneth_S._Davis'},
     {'name': 'R.H.C. Davis',
      'url': 'https://en.wikipedia.org/wiki/Ralph_Henry_Carless_Davis'},
     {'name': 'David Day',
      'url': 'https://en.wikipedia.org/wiki/David_Day_(historian)'},
     {'name': 'Renzo De Felice',
      'url': 'https://en.wikipedia.org/wiki/Renzo_De_Felice'},
     {'name': 'Len Deighton', 'url': 'https://en.wikipedia.org/wiki/Len_Deighton'},
     {'name': 'Carl N. Degler',
      'url': 'https://en.wikipedia.org/wiki/Carl_N._Degler'},
     {'name': 'Esther Delisle',
      'url': 'https://en.wikipedia.org/wiki/Esther_Delisle'},
     {'name': 'Jean Delumeau',
      'url': 'https://en.wikipedia.org/wiki/Jean_Delumeau'},
     {'name': 'Marcel Detienne',
      'url': 'https://en.wikipedia.org/wiki/Marcel_Detienne'},
     {'name': 'Alexandre Deulofeu',
      'url': 'https://en.wikipedia.org/wiki/Alexandre_Deulofeu'},
     {'name': 'Isaac Deutscher',
      'url': 'https://en.wikipedia.org/wiki/Isaac_Deutscher'},
     {'name': 'Tom M. Devine',
      'url': 'https://en.wikipedia.org/wiki/Tom_M._Devine'},
     {'name': 'Wu Di',
      'url': 'https://en.wikipedia.org/wiki/Wu_Di_(film_critic_and_historian)'},
     {'name': 'Igor M. Diakonov',
      'url': 'https://en.wikipedia.org/wiki/Igor_M._Diakonov'},
     {'name': 'David Herbert Donald',
      'url': 'https://en.wikipedia.org/wiki/David_Herbert_Donald'},
     {'name': 'Gordon Donaldson',
      'url': 'https://en.wikipedia.org/wiki/Gordon_Donaldson'},
     {'name': 'Susan Doran', 'url': 'https://en.wikipedia.org/wiki/Susan_Doran'},
     {'name': 'William Doyle',
      'url': 'https://en.wikipedia.org/wiki/William_Doyle_(historian)'},
     {'name': 'Georges Duby', 'url': 'https://en.wikipedia.org/wiki/Georges_Duby'},
     {'name': 'William S. Dudley',
      'url': 'https://en.wikipedia.org/wiki/William_S._Dudley'},
     {'name': 'Robert Dudley Edwards',
      'url': 'https://en.wikipedia.org/wiki/Robert_Dudley_Edwards'},
     {'name': 'Eamon Duffy', 'url': 'https://en.wikipedia.org/wiki/Eamon_Duffy'},
     {'name': 'A. Hunter Dupree',
      'url': 'https://en.wikipedia.org/wiki/A._Hunter_Dupree'},
     {'name': 'Trevor Dupuy', 'url': 'https://en.wikipedia.org/wiki/Trevor_Dupuy'},
     {'name': 'Jean-Baptiste Duroselle',
      'url': 'https://en.wikipedia.org/wiki/Jean-Baptiste_Duroselle'},
     {'name': 'Harold James Dyos',
      'url': 'https://en.wikipedia.org/wiki/Harold_James_Dyos'},
     {'name': 'Elizabeth Eisenstein',
      'url': 'https://en.wikipedia.org/wiki/Elizabeth_Eisenstein'},
     {'name': 'Geoff Eley', 'url': 'https://en.wikipedia.org/wiki/Geoff_Eley'},
     {'name': 'John Elliott',
      'url': 'https://en.wikipedia.org/wiki/John_Elliott_(historian)'},
     {'name': 'Joseph J. Ellis',
      'url': 'https://en.wikipedia.org/wiki/Joseph_J._Ellis'},
     {'name': 'Geoffrey Elton',
      'url': 'https://en.wikipedia.org/wiki/Geoffrey_Elton'},
     {'name': 'Peter Englund',
      'url': 'https://en.wikipedia.org/wiki/Peter_Englund'},
     {'name': 'Robert Malcolm Errington',
      'url': 'https://en.wikipedia.org/wiki/Robert_Malcolm_Errington'},
     {'name': 'Richard J. Evans',
      'url': 'https://en.wikipedia.org/wiki/Richard_J._Evans'},
     {'name': 'Alf Evers', 'url': 'https://en.wikipedia.org/wiki/Alf_Evers'},
     {'name': 'Brian Farrell',
      'url': 'https://en.wikipedia.org/wiki/Brian_Farrell_(broadcaster)'},
     {'name': 'John Lister Illingworth Fennell',
      'url': 'https://en.wikipedia.org/wiki/John_Lister_Illingworth_Fennell'},
     {'name': 'Niall Ferguson',
      'url': 'https://en.wikipedia.org/wiki/Niall_Ferguson'},
     {'name': u'Bo\u017eidar Ferjan\u010di\u0107',
      'url': 'https://en.wikipedia.org/wiki/Bo%C5%BEidar_Ferjan%C4%8Di%C4%87'},
     {'name': 'Marc Ferro', 'url': 'https://en.wikipedia.org/wiki/Marc_Ferro'},
     {'name': 'Joachim Fest', 'url': 'https://en.wikipedia.org/wiki/Joachim_Fest'},
     {'name': 'David Feuerwerker',
      'url': 'https://en.wikipedia.org/wiki/David_Feuerwerker'},
     {'name': 'Heinrich Fichtenau',
      'url': 'https://en.wikipedia.org/wiki/Heinrich_Fichtenau'},
     {'name': 'David Kenneth Fieldhouse',
      'url': 'https://en.wikipedia.org/wiki/David_Kenneth_Fieldhouse'},
     {'name': 'Orlando Figes',
      'url': 'https://en.wikipedia.org/wiki/Orlando_Figes'},
     {'name': 'Robert O. Fink',
      'url': 'https://en.wikipedia.org/wiki/Robert_O._Fink'},
     {'name': 'Moses Finley', 'url': 'https://en.wikipedia.org/wiki/Moses_Finley'},
     {'name': 'David Hackett Fischer',
      'url': 'https://en.wikipedia.org/wiki/David_Hackett_Fischer'},
     {'name': 'Fritz Fischer',
      'url': 'https://en.wikipedia.org/wiki/Fritz_Fischer_(historian)'},
     {'name': 'Frances FitzGerald',
      'url': 'https://en.wikipedia.org/wiki/Frances_FitzGerald_(journalist)'},
     {'name': 'Judith Flanders',
      'url': 'https://en.wikipedia.org/wiki/Judith_Flanders'},
     {'name': 'Robert Fogel', 'url': 'https://en.wikipedia.org/wiki/Robert_Fogel'},
     {'name': 'Eric Foner', 'url': 'https://en.wikipedia.org/wiki/Eric_Foner'},
     {'name': 'Shelby Foote', 'url': 'https://en.wikipedia.org/wiki/Shelby_Foote'},
     {'name': 'Amanda Foreman',
      'url': 'https://en.wikipedia.org/wiki/Amanda_Foreman_(historian)'},
     {'name': 'Michel Foucault',
      'url': 'https://en.wikipedia.org/wiki/Michel_Foucault'},
     {'name': 'Jo Fox', 'url': 'https://en.wikipedia.org/wiki/Jo_Fox'},
     {'name': 'Robin Lane Fox',
      'url': 'https://en.wikipedia.org/wiki/Robin_Lane_Fox'},
     {'name': 'Stephen Fox',
      'url': 'https://en.wikipedia.org/wiki/Stephen_Fox_(author/educator)'},
     {'name': 'Elizabeth Fox-Genovese',
      'url': 'https://en.wikipedia.org/wiki/Elizabeth_Fox-Genovese'},
     {'name': 'Walter Frank', 'url': 'https://en.wikipedia.org/wiki/Walter_Frank'},
     {'name': 'H. Bruce Franklin',
      'url': 'https://en.wikipedia.org/wiki/H._Bruce_Franklin'},
     {'name': 'Antonia Fraser',
      'url': 'https://en.wikipedia.org/wiki/Antonia_Fraser'},
     {'name': 'Frank Freidel',
      'url': 'https://en.wikipedia.org/wiki/Frank_Freidel'},
     {'name': 'Joseph Friedenson',
      'url': 'https://en.wikipedia.org/wiki/Joseph_Friedenson'},
     {'name': 'Henry Friedlander',
      'url': 'https://en.wikipedia.org/wiki/Henry_Friedlander'},
     {'name': u'Saul Friedl\xe4nder',
      'url': 'https://en.wikipedia.org/wiki/Saul_Friedl%C3%A4nder'},
     {'name': 'Sheppard Frere',
      'url': 'https://en.wikipedia.org/wiki/Sheppard_Frere'},
     {'name': 'David Fromkin',
      'url': 'https://en.wikipedia.org/wiki/David_Fromkin'},
     {'name': 'Bruno Fuligni',
      'url': 'https://en.wikipedia.org/wiki/Bruno_Fuligni'},
     {'name': 'Francis Fukuyama',
      'url': 'https://en.wikipedia.org/wiki/Francis_Fukuyama'},
     {'name': u'Fran\xe7ois Furet',
      'url': 'https://en.wikipedia.org/wiki/Fran%C3%A7ois_Furet'},
     {'name': 'Femme Gaastra',
      'url': 'https://en.wikipedia.org/wiki/Femme_Gaastra'},
     {'name': 'John Lewis Gaddis',
      'url': 'https://en.wikipedia.org/wiki/John_Lewis_Gaddis'},
     {'name': 'Lloyd Gardner',
      'url': 'https://en.wikipedia.org/wiki/Lloyd_Gardner'},
     {'name': 'Peter Gay', 'url': 'https://en.wikipedia.org/wiki/Peter_Gay'},
     {'name': 'Eugene Genovese',
      'url': 'https://en.wikipedia.org/wiki/Eugene_Genovese'},
     {'name': u'Fran\xe7ois G\xe9r\xe9',
      'url': 'https://en.wikipedia.org/wiki/Fran%C3%A7ois_G%C3%A9r%C3%A9'},
     {'name': 'Imanuel Geiss',
      'url': 'https://en.wikipedia.org/wiki/Imanuel_Geiss'},
     {'name': 'Christian Gerlach',
      'url': 'https://en.wikipedia.org/wiki/Christian_Gerlach'},
     {'name': 'N.H. Gibbs', 'url': 'https://en.wikipedia.org/wiki/N.H._Gibbs'},
     {'name': 'William Gibson',
      'url': 'https://en.wikipedia.org/wiki/William_Gibson_(historian)'},
     {'name': 'Martin Gilbert',
      'url': 'https://en.wikipedia.org/wiki/Martin_Gilbert'},
     {'name': 'Carlo Ginzburg',
      'url': 'https://en.wikipedia.org/wiki/Carlo_Ginzburg'},
     {'name': 'Jan Glete', 'url': 'https://en.wikipedia.org/wiki/Jan_Glete'},
     {'name': 'Eric F. Goldman',
      'url': 'https://en.wikipedia.org/wiki/Eric_F._Goldman'},
     {'name': 'James Goldrick',
      'url': 'https://en.wikipedia.org/wiki/James_Goldrick'},
     {'name': 'Adrian Goldsworthy',
      'url': 'https://en.wikipedia.org/wiki/Adrian_Goldsworthy'},
     {'name': 'Brison D. Gooch',
      'url': 'https://en.wikipedia.org/wiki/Brison_D._Gooch'},
     {'name': 'Doris Kearns Goodwin',
      'url': 'https://en.wikipedia.org/wiki/Doris_Kearns_Goodwin'},
     {'name': 'Andrew Gordon',
      'url': 'https://en.wikipedia.org/wiki/Andrew_Gordon_(naval_historian)'},
     {'name': 'Gerald S. Graham',
      'url': 'https://en.wikipedia.org/wiki/Gerald_S._Graham'},
     {'name': 'Jack Granatstein',
      'url': 'https://en.wikipedia.org/wiki/Jack_Granatstein'},
     {'name': 'Peter Green',
      'url': 'https://en.wikipedia.org/wiki/Peter_Green_(historian)'},
     {'name': 'Vivian H.H. Green',
      'url': 'https://en.wikipedia.org/wiki/Rev._Vivian_Green'},
     {'name': 'John Robert Greene',
      'url': 'https://en.wikipedia.org/wiki/John_Robert_Greene'},
     {'name': 'Roger D. Griffin',
      'url': 'https://en.wikipedia.org/wiki/Roger_D._Griffin'},
     {'name': 'Ranajit Guha', 'url': 'https://en.wikipedia.org/wiki/Ranajit_Guha'},
     {'name': 'Ramchandra Guha',
      'url': 'https://en.wikipedia.org/wiki/Ramchandra_Guha'},
     {'name': 'Lev Gumilyov', 'url': 'https://en.wikipedia.org/wiki/Lev_Gumilyov'},
     {'name': 'Oliver Gurney',
      'url': 'https://en.wikipedia.org/wiki/Oliver_Gurney'},
     {'name': 'John Guy',
      'url': 'https://en.wikipedia.org/wiki/John_Guy_(historian)'},
     {'name': 'Irfan Habib', 'url': 'https://en.wikipedia.org/wiki/Irfan_Habib'},
     {'name': 'Sheldon Hackney',
      'url': 'https://en.wikipedia.org/wiki/Sheldon_Hackney'},
     {'name': 'Kenneth J. Hagan',
      'url': 'https://en.wikipedia.org/wiki/Kenneth_J._Hagan'},
     {'name': 'Claude Hall', 'url': 'https://en.wikipedia.org/wiki/Claude_Hall'},
     {'name': 'John Whitney Hall',
      'url': 'https://en.wikipedia.org/wiki/John_Whitney_Hall'},
     {'name': 'Bruce Barrymore Halpenny',
      'url': 'https://en.wikipedia.org/wiki/Bruce_Barrymore_Halpenny'},
     {'name': 'N. G. L. Hammond',
      'url': 'https://en.wikipedia.org/wiki/N._G._L._Hammond'},
     {'name': 'Victor Davis Hanson',
      'url': 'https://en.wikipedia.org/wiki/Victor_Davis_Hanson'},
     {'name': 'Syed Nomanul Haq',
      'url': 'https://en.wikipedia.org/wiki/Syed_Nomanul_Haq'},
     {'name': 'Dick Harrison',
      'url': 'https://en.wikipedia.org/wiki/Dick_Harrison'},
     {'name': 'Peter Harrison',
      'url': 'https://en.wikipedia.org/wiki/Peter_Harrison_(historian)'},
     {'name': 'Max Hastings', 'url': 'https://en.wikipedia.org/wiki/Max_Hastings'},
     {'name': 'John Hattendorf',
      'url': 'https://en.wikipedia.org/wiki/John_Hattendorf'},
     {'name': 'Ragnhild Hatton',
      'url': 'https://en.wikipedia.org/wiki/Ragnhild_Hatton'},
     {'name': 'Denys Hay', 'url': 'https://en.wikipedia.org/wiki/Denys_Hay'},
     {'name': 'John Daniel Hayes',
      'url': 'https://en.wikipedia.org/wiki/John_Daniel_Hayes'},
     {'name': 'Ingo Heidbrink',
      'url': 'https://en.wikipedia.org/wiki/Ingo_Heidbrink'},
     {'name': 'Jeffrey Herf', 'url': 'https://en.wikipedia.org/wiki/Jeffrey_Herf'},
     {'name': 'Arthur Herman',
      'url': 'https://en.wikipedia.org/wiki/Arthur_Herman'},
     {'name': 'Michael Hicks',
      'url': 'https://en.wikipedia.org/wiki/Michael_Hicks_(historian)'},
     {'name': 'Raul Hilberg', 'url': 'https://en.wikipedia.org/wiki/Raul_Hilberg'},
     {'name': 'Klaus Hildebrand',
      'url': 'https://en.wikipedia.org/wiki/Klaus_Hildebrand'},
     {'name': 'Christopher Hill',
      'url': 'https://en.wikipedia.org/wiki/Christopher_Hill_(historian)'},
     {'name': 'Andreas Hillgruber',
      'url': 'https://en.wikipedia.org/wiki/Andreas_Hillgruber'},
     {'name': 'Richard L. Hills',
      'url': 'https://en.wikipedia.org/wiki/Richard_L._Hills'},
     {'name': 'Gertrude Himmelfarb',
      'url': 'https://en.wikipedia.org/wiki/Gertrude_Himmelfarb'},
     {'name': 'Harry Hinsley',
      'url': 'https://en.wikipedia.org/wiki/Harry_Hinsley'},
     {'name': 'Eric Hobsbawm',
      'url': 'https://en.wikipedia.org/wiki/Eric_Hobsbawm'},
     {'name': 'Marshall Hodgson',
      'url': 'https://en.wikipedia.org/wiki/Marshall_Hodgson'},
     {'name': 'Richard Hofstadter',
      'url': 'https://en.wikipedia.org/wiki/Richard_Hofstadter'},
     {'name': 'Peter Hoffmann',
      'url': 'https://en.wikipedia.org/wiki/Peter_Hoffmann_(historian)'},
     {'name': 'David Hoggan', 'url': 'https://en.wikipedia.org/wiki/David_Hoggan'},
     {'name': 'Hajo Holborn', 'url': 'https://en.wikipedia.org/wiki/Hajo_Holborn'},
     {'name': 'Tom Holland',
      'url': 'https://en.wikipedia.org/wiki/Tom_Holland_(author)'},
     {'name': 'C. Warren Hollister',
      'url': 'https://en.wikipedia.org/wiki/C._Warren_Hollister'},
     {'name': 'George Holmes (professor)',
      'url': 'https://en.wikipedia.org/wiki/George_Holmes_(professor)'},
     {'name': 'Richard Holmes',
      'url': 'https://en.wikipedia.org/wiki/Richard_Holmes_(historian)'},
     {'name': 'Ed Hooper', 'url': 'https://en.wikipedia.org/wiki/Ed_Hooper'},
     {'name': 'A.G. Hopkins', 'url': 'https://en.wikipedia.org/wiki/A.G._Hopkins'},
     {'name': 'Keith Hopkins',
      'url': 'https://en.wikipedia.org/wiki/Keith_hopkins'},
     {'name': 'Albert Hourani',
      'url': 'https://en.wikipedia.org/wiki/Albert_Hourani'},
     {'name': 'Youssef Hourany',
      'url': 'https://en.wikipedia.org/wiki/Youssef_Hourany'},
     {'name': 'Daniel Horowitz',
      'url': 'https://en.wikipedia.org/wiki/Daniel_Horowitz'},
     {'name': 'Helen Lefkowitz Horowitz',
      'url': 'https://en.wikipedia.org/wiki/Helen_Lefkowitz_Horowitz'},
     {'name': 'Michiel Horn', 'url': 'https://en.wikipedia.org/wiki/Michiel_Horn'},
     {'name': 'Alistair Horne',
      'url': 'https://en.wikipedia.org/wiki/Alistair_Horne'},
     {'name': 'Michael Howard',
      'url': 'https://en.wikipedia.org/wiki/Michael_Howard_(historian)'},
     {'name': 'Robert Hughes',
      'url': 'https://en.wikipedia.org/wiki/Robert_Hughes_(critic)'},
     {'name': 'Andrew Hunt',
      'url': 'https://en.wikipedia.org/wiki/Andrew_Hunt_(historian)'},
     {'name': 'Tristram Hunt',
      'url': 'https://en.wikipedia.org/wiki/Tristram_Hunt'},
     {'name': 'Mark C. Hunter',
      'url': 'https://en.wikipedia.org/wiki/Mark_C._Hunter'},
     {'name': 'Halil Inalcik',
      'url': 'https://en.wikipedia.org/wiki/Halil_Inalcik'},
     {'name': 'Iqbal, Sheikh Mohammad',
      'url': 'https://en.wikipedia.org/wiki/Sheikh_Mohammad_Iqbal'},
     {'name': 'Jonathan Israel',
      'url': 'https://en.wikipedia.org/wiki/Jonathan_Israel'},
     {'name': u'Eberhard J\xe4ckel',
      'url': 'https://en.wikipedia.org/wiki/Eberhard_J%C3%A4ckel'},
     {'name': 'Julian T. Jackson',
      'url': 'https://en.wikipedia.org/wiki/Julian_T._Jackson'},
     {'name': 'Harold James',
      'url': 'https://en.wikipedia.org/wiki/Harold_James_(historian)'},
     {'name': 'Nikoloz Janashia',
      'url': 'https://en.wikipedia.org/wiki/Nikoloz_Janashia'},
     {'name': 'Simon Janashia',
      'url': 'https://en.wikipedia.org/wiki/Simon_Janashia'},
     {'name': 'Marius Jansen',
      'url': 'https://en.wikipedia.org/wiki/Marius_Jansen'},
     {'name': 'Pawel Jasienica',
      'url': 'https://en.wikipedia.org/wiki/Pawel_Jasienica'},
     {'name': 'Merrill Jensen',
      'url': 'https://en.wikipedia.org/wiki/Merrill_Jensen_(historian)'},
     {'name': 'Richard J. Jensen',
      'url': 'https://en.wikipedia.org/wiki/Richard_J._Jensen'},
     {'name': 'Khasnor Johan',
      'url': 'https://en.wikipedia.org/wiki/Khasnor_Johan'},
     {'name': 'Paul Johnson',
      'url': 'https://en.wikipedia.org/wiki/Paul_Johnson_(writer)'},
     {'name': 'Robert Erwin Johnson',
      'url': 'https://en.wikipedia.org/wiki/Robert_Erwin_Johnson'},
     {'name': 'Mauno Jokipii',
      'url': 'https://en.wikipedia.org/wiki/Mauno_Jokipii'},
     {'name': 'A.H.M. Jones',
      'url': 'https://en.wikipedia.org/wiki/Arnold_Hugh_Martin_Jones'},
     {'name': 'Gwyn Jones',
      'url': 'https://en.wikipedia.org/wiki/Gwyn_Jones_(author)'},
     {'name': 'George Hilton Jones III',
      'url': 'https://en.wikipedia.org/wiki/George_Hilton_Jones,_III_(historian)'},
     {'name': 'Loe de Jong', 'url': 'https://en.wikipedia.org/wiki/Loe_de_Jong'},
     {'name': 'Tony Judt', 'url': 'https://en.wikipedia.org/wiki/Tony_Judt'},
     {'name': 'David S. Katz',
      'url': 'https://en.wikipedia.org/wiki/David_S._Katz'},
     {'name': 'Donald Kagan', 'url': 'https://en.wikipedia.org/wiki/Donald_Kagan'},
     {'name': 'Elie Kedourie',
      'url': 'https://en.wikipedia.org/wiki/Elie_Kedourie'},
     {'name': 'Rod Kedward', 'url': 'https://en.wikipedia.org/wiki/Rod_Kedward'},
     {'name': 'John Keegan', 'url': 'https://en.wikipedia.org/wiki/John_Keegan'},
     {'name': 'Nushiravan Keihanizadeh',
      'url': 'https://en.wikipedia.org/wiki/Nushiravan_Keihanizadeh'},
     {'name': 'John H. Kemble',
      'url': 'https://en.wikipedia.org/wiki/John_H._Kemble'},
     {'name': 'Paul Murray Kendall',
      'url': 'https://en.wikipedia.org/wiki/Paul_Murray_Kendall'},
     {'name': 'Elizabeth Topham Kennan',
      'url': 'https://en.wikipedia.org/wiki/Elizabeth_Topham_Kennan'},
     {'name': 'George F. Kennan',
      'url': 'https://en.wikipedia.org/wiki/George_F._Kennan'},
     {'name': 'James Kennedy',
      'url': 'https://en.wikipedia.org/wiki/James_Kennedy_(historian)'},
     {'name': 'Paul Kennedy', 'url': 'https://en.wikipedia.org/wiki/Paul_Kennedy'},
     {'name': 'W. Hudson Kensel',
      'url': 'https://en.wikipedia.org/wiki/W._Hudson_Kensel'},
     {'name': 'Ian Kershaw', 'url': 'https://en.wikipedia.org/wiki/Ian_Kershaw'},
     {'name': 'Daniel J. Kevles',
      'url': 'https://en.wikipedia.org/wiki/Daniel_J._Kevles'},
     {'name': 'Khan Roshan Khan',
      'url': 'https://en.wikipedia.org/wiki/Khan_Roshan_Khan'},
     {'name': 'Kim Jung-bae', 'url': 'https://en.wikipedia.org/wiki/Kim_Jung-bae'},
     {'name': 'Michael King', 'url': 'https://en.wikipedia.org/wiki/Michael_King'},
     {'name': 'Patrick Kinross',
      'url': 'https://en.wikipedia.org/wiki/Patrick_Kinross'},
     {'name': 'Henry Kissinger',
      'url': 'https://en.wikipedia.org/wiki/Henry_Kissinger'},
     {'name': 'Martin Kitchen',
      'url': 'https://en.wikipedia.org/wiki/Martin_Kitchen'},
     {'name': 'Simon Kitson', 'url': 'https://en.wikipedia.org/wiki/Simon_Kitson'},
     {'name': 'Matti Klinge', 'url': 'https://en.wikipedia.org/wiki/Matti_Klinge'},
     {'name': 'Felix Klos', 'url': 'https://en.wikipedia.org/wiki/Felix_Klos'},
     {'name': 'R.J.B. Knight',
      'url': 'https://en.wikipedia.org/wiki/R.J.B._Knight'},
     {'name': 'Yuri Knorozov',
      'url': 'https://en.wikipedia.org/wiki/Yuri_Knorozov'},
     {'name': 'Eberhard Kolb',
      'url': 'https://en.wikipedia.org/wiki/Eberhard_Kolb'},
     {'name': 'Gabriel Kolko',
      'url': 'https://en.wikipedia.org/wiki/Gabriel_Kolko'},
     {'name': 'Claudia Koonz',
      'url': 'https://en.wikipedia.org/wiki/Claudia_Koonz'},
     {'name': 'Andrey Korotayev',
      'url': 'https://en.wikipedia.org/wiki/Andrey_Korotayev'},
     {'name': 'Ernst Kossmann',
      'url': 'https://en.wikipedia.org/wiki/Ernst_Kossmann'},
     {'name': 'Philip A. Kuhn',
      'url': 'https://en.wikipedia.org/wiki/Philip_A._Kuhn'},
     {'name': 'Thomas Kuhn', 'url': 'https://en.wikipedia.org/wiki/Thomas_Kuhn'},
     {'name': 'Myoma Myint Kywe',
      'url': 'https://en.wikipedia.org/wiki/Myoma_Myint_Kywe'},
     {'name': 'K.S. Lal', 'url': 'https://en.wikipedia.org/wiki/K.S._Lal'},
     {'name': 'Benjamin Woods Labaree',
      'url': 'https://en.wikipedia.org/wiki/Benjamin_Woods_Labaree'},
     {'name': 'Brij Lal',
      'url': 'https://en.wikipedia.org/wiki/Brij_Lal_(historian)'},
     {'name': 'Abdallah Laroui',
      'url': 'https://en.wikipedia.org/wiki/Abdallah_Laroui'},
     {'name': 'Leopold Labedz',
      'url': 'https://en.wikipedia.org/wiki/Leopold_Labedz'},
     {'name': 'Andrew Lambert',
      'url': 'https://en.wikipedia.org/wiki/Andrew_Lambert'},
     {'name': 'Harold Lamb', 'url': 'https://en.wikipedia.org/wiki/Harold_Lamb'},
     {'name': 'Ricardo Lancaster-Jones y Verea',
      'url': 'https://en.wikipedia.org/wiki/Ricardo_Lancaster-Jones_y_Verea'},
     {'name': 'David Lavender',
      'url': 'https://en.wikipedia.org/wiki/David_Lavender'},
     {'name': 'Walter LaFeber',
      'url': 'https://en.wikipedia.org/wiki/Walter_LaFeber'},
     {'name': 'Daniel Leab', 'url': 'https://en.wikipedia.org/wiki/Daniel_Leab'},
     {'name': 'Jacques Le Goff',
      'url': 'https://en.wikipedia.org/wiki/Jacques_Le_Goff'},
     {'name': 'Robert Leckie',
      'url': 'https://en.wikipedia.org/wiki/Robert_Leckie_(author)'},
     {'name': 'William Leuchtenburg',
      'url': 'https://en.wikipedia.org/wiki/William_Leuchtenburg'},
     {'name': 'Barbara Levick',
      'url': 'https://en.wikipedia.org/wiki/Barbara_Levick'},
     {'name': 'David Levering Lewis',
      'url': 'https://en.wikipedia.org/wiki/David_Levering_Lewis'},
     {'name': 'Emmanuel Le Roy Ladurie',
      'url': 'https://en.wikipedia.org/wiki/Emmanuel_Le_Roy_Ladurie'},
     {'name': 'Lee Ki-baek', 'url': 'https://en.wikipedia.org/wiki/Lee_Ki-baek'},
     {'name': 'Li Ao', 'url': 'https://en.wikipedia.org/wiki/Li_Ao'},
     {'name': 'Leon F. Litwack',
      'url': 'https://en.wikipedia.org/wiki/Leon_F._Litwack'},
     {'name': 'Xinru Liu', 'url': 'https://en.wikipedia.org/wiki/Xinru_Liu'},
     {'name': 'Mario Liverani',
      'url': 'https://en.wikipedia.org/wiki/Mario_Liverani'},
     {'name': u'Rado\u0161 Lju\u0161i\u0107',
      'url': 'https://en.wikipedia.org/wiki/Rado%C5%A1_Lju%C5%A1i%C4%87'},
     {'name': 'David Loades', 'url': 'https://en.wikipedia.org/wiki/David_Loades'},
     {'name': 'James W. Loewen',
      'url': 'https://en.wikipedia.org/wiki/James_W._Loewen'},
     {'name': 'Elizabeth Longford',
      'url': 'https://en.wikipedia.org/wiki/Elizabeth_Pakenham,_Countess_of_Longford'},
     {'name': u'Erik L\xf6nnroth',
      'url': 'https://en.wikipedia.org/wiki/Erik_L%C3%B6nnroth'},
     {'name': 'Walter Lord', 'url': 'https://en.wikipedia.org/wiki/Walter_Lord'},
     {'name': 'John Lukacs', 'url': 'https://en.wikipedia.org/wiki/John_Lukacs'},
     {'name': 'Charles B. MacDonald',
      'url': 'https://en.wikipedia.org/wiki/Charles_B._MacDonald'},
     {'name': 'Stuart Macintyre',
      'url': 'https://en.wikipedia.org/wiki/Stuart_Macintyre'},
     {'name': 'Forrest McDonald',
      'url': 'https://en.wikipedia.org/wiki/Forrest_McDonald'},
     {'name': 'K. B. McFarlane',
      'url': 'https://en.wikipedia.org/wiki/K._B._McFarlane'},
     {'name': 'Ross McKibbin',
      'url': 'https://en.wikipedia.org/wiki/Ross_McKibbin'},
     {'name': 'Rosamond McKitterick',
      'url': 'https://en.wikipedia.org/wiki/Rosamond_McKitterick'},
     {'name': 'Margaret MacMillan',
      'url': 'https://en.wikipedia.org/wiki/Margaret_MacMillan'},
     {'name': 'William Miller Macmillan',
      'url': 'https://en.wikipedia.org/wiki/William_Miller_Macmillan'},
     {'name': 'Ramsay MacMullen',
      'url': 'https://en.wikipedia.org/wiki/Ramsay_MacMullen'},
     {'name': 'Magnus Magnusson',
      'url': 'https://en.wikipedia.org/wiki/Magnus_Magnusson'},
     {'name': 'Piers Mackesy',
      'url': 'https://en.wikipedia.org/wiki/Piers_Mackesy'},
     {'name': 'Leonard Maltin',
      'url': 'https://en.wikipedia.org/wiki/Leonard_Maltin'},
     {'name': 'Charles S. Maier',
      'url': 'https://en.wikipedia.org/wiki/Charles_S._Maier'},
     {'name': 'Paul L. Maier',
      'url': 'https://en.wikipedia.org/wiki/Paul_L._Maier'},
     {'name': 'Pauline Maier',
      'url': 'https://en.wikipedia.org/wiki/Pauline_Maier'},
     {'name': 'William Manchester',
      'url': 'https://en.wikipedia.org/wiki/William_Manchester'},
     {'name': 'Adel Manna', 'url': 'https://en.wikipedia.org/wiki/Adel_Manna'},
     {'name': 'Golo Mann', 'url': 'https://en.wikipedia.org/wiki/Golo_Mann'},
     {'name': 'Susan Mann', 'url': 'https://en.wikipedia.org/wiki/Susan_Mann'},
     {'name': 'Robert Mann', 'url': 'https://en.wikipedia.org/wiki/Robert_Mann'},
     {'name': 'Philip Mansel',
      'url': 'https://en.wikipedia.org/wiki/Philip_Mansel'},
     {'name': 'Arthur Marder',
      'url': 'https://en.wikipedia.org/wiki/Arthur_Marder'},
     {'name': 'Timothy Mason',
      'url': 'https://en.wikipedia.org/wiki/Timothy_Mason'},
     {'name': 'Henri-Jean Martin',
      'url': 'https://en.wikipedia.org/wiki/Henri-Jean_Martin'},
     {'name': 'Rev. F.X. Martin',
      'url': 'https://en.wikipedia.org/wiki/F.X._Martin'},
     {'name': 'Michael Marrus',
      'url': 'https://en.wikipedia.org/wiki/Michael_Marrus'},
     {'name': 'Mark Mazower', 'url': 'https://en.wikipedia.org/wiki/Mark_Mazower'},
     {'name': 'David McCullough',
      'url': 'https://en.wikipedia.org/wiki/David_McCullough'},
     {'name': 'William S. McFeely',
      'url': 'https://en.wikipedia.org/wiki/William_S._McFeely'},
     {'name': 'James M. McPherson',
      'url': 'https://en.wikipedia.org/wiki/James_M._McPherson'},
     {'name': 'William McNeill',
      'url': 'https://en.wikipedia.org/wiki/William_Hardy_McNeill'},
     {'name': 'Laurence Marvin',
      'url': 'https://en.wikipedia.org/wiki/Laurence_Marvin'},
     {'name': 'Garrett Mattingly',
      'url': 'https://en.wikipedia.org/wiki/Garrett_Mattingly'},
     {'name': 'Arno J. Mayer',
      'url': 'https://en.wikipedia.org/wiki/Arno_J._Mayer'},
     {'name': 'Richard Maybury',
      'url': 'https://en.wikipedia.org/wiki/Richard_J._Maybury'},
     {'name': 'Neil McKendrick',
      'url': 'https://en.wikipedia.org/wiki/Neil_McKendrick'},
     {'name': 'D. W. Meinig', 'url': 'https://en.wikipedia.org/wiki/D._W._Meinig'},
     {'name': 'Evaldo Cabral de Mello',
      'url': 'https://en.wikipedia.org/wiki/Evaldo_Cabral_de_Mello'},
     {'name': 'Russell Menard',
      'url': 'https://en.wikipedia.org/wiki/Russell_Menard'},
     {'name': 'Thomas C. Mendenhall',
      'url': 'https://en.wikipedia.org/wiki/Thomas_C._Mendenhall_(historian)'},
     {'name': 'Josef W. Meri',
      'url': 'https://en.wikipedia.org/wiki/Josef_W._Meri'},
     {'name': 'Barbara Metcalf',
      'url': 'https://en.wikipedia.org/wiki/Barbara_Metcalf'},
     {'name': u'Rade Mihalj\u010di\u0107',
      'url': 'https://en.wikipedia.org/wiki/Rade_Mihalj%C4%8Di%C4%87'},
     {'name': 'Perry Miller', 'url': 'https://en.wikipedia.org/wiki/Perry_Miller'},
     {'name': 'Giles Milton', 'url': 'https://en.wikipedia.org/wiki/Giles_Milton'},
     {'name': u'Zora Mintalov\xe1 \u2013 Zubercov\xe1',
      'url': 'https://en.wikipedia.org/wiki/Zora_Mintalov%C3%A1_%E2%80%93_Zubercov%C3%A1'},
     {'name': 'Yagutil Mishiev',
      'url': 'https://en.wikipedia.org/wiki/Yagutil_Mishiev'},
     {'name': 'Hans Mommsen', 'url': 'https://en.wikipedia.org/wiki/Hans_Mommsen'},
     {'name': 'Wolfgang Mommsen',
      'url': 'https://en.wikipedia.org/wiki/Wolfgang_Mommsen'},
     {'name': 'Simon Sebag Montefiore',
      'url': 'https://en.wikipedia.org/wiki/Simon_Sebag_Montefiore'},
     {'name': 'Theodore William Moody',
      'url': 'https://en.wikipedia.org/wiki/Theodore_William_Moody'},
     {'name': 'Edmund Morgan',
      'url': 'https://en.wikipedia.org/wiki/Edmund_Morgan_(historian)'},
     {'name': 'Kenneth O. Morgan',
      'url': 'https://en.wikipedia.org/wiki/Kenneth_O._Morgan'},
     {'name': 'William J. Morgan (historian)',
      'url': 'https://en.wikipedia.org/wiki/William_J._Morgan_(historian)'},
     {'name': 'Samuel Eliot Morison',
      'url': 'https://en.wikipedia.org/wiki/Samuel_Eliot_Morison'},
     {'name': 'Benny Morris', 'url': 'https://en.wikipedia.org/wiki/Benny_Morris'},
     {'name': 'Ian Mortimer',
      'url': 'https://en.wikipedia.org/wiki/Ian_Mortimer_(historian)'},
     {'name': 'W.L. Morton', 'url': 'https://en.wikipedia.org/wiki/W.L._Morton'},
     {'name': 'George Mosse', 'url': 'https://en.wikipedia.org/wiki/George_Mosse'},
     {'name': 'Roland Mousnier',
      'url': 'https://en.wikipedia.org/wiki/Roland_Mousnier'},
     {'name': 'Mubarak Ali', 'url': 'https://en.wikipedia.org/wiki/Mubarak_Ali'},
     {'name': 'Joseph Needham',
      'url': 'https://en.wikipedia.org/wiki/Joseph_Needham'},
     {'name': 'Cynthia Neville',
      'url': 'https://en.wikipedia.org/wiki/Cynthia_Neville'},
     {'name': 'Leo Niehorster',
      'url': 'https://en.wikipedia.org/wiki/Leo_Niehorster'},
     {'name': 'Thomas Nipperdey',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Nipperdey'},
     {'name': 'Ernst Nolte', 'url': 'https://en.wikipedia.org/wiki/Ernst_Nolte'},
     {'name': u'Stojan Novakovi\u0107',
      'url': 'https://en.wikipedia.org/wiki/Stojan_Novakovi%C4%87'},
     {'name': "Robin O'Neil",
      'url': 'https://en.wikipedia.org/wiki/Robin_O%27Neil'},
     {'name': 'Josiah Ober', 'url': 'https://en.wikipedia.org/wiki/Josiah_Ober'},
     {'name': 'Heiko Oberman',
      'url': 'https://en.wikipedia.org/wiki/Heiko_Oberman'},
     {'name': 'W. H. Oliver', 'url': 'https://en.wikipedia.org/wiki/W._H._Oliver'},
     {'name': 'Michael Oren', 'url': 'https://en.wikipedia.org/wiki/Michael_Oren'},
     {'name': 'Margaret Ormsby',
      'url': 'https://en.wikipedia.org/wiki/Margaret_Ormsby'},
     {'name': u'\u0130lber Ortayl\u0131',
      'url': 'https://en.wikipedia.org/wiki/%C4%B0lber_Ortayl%C4%B1'},
     {'name': 'Fernand Ouellet',
      'url': 'https://en.wikipedia.org/wiki/Fernand_Ouellet'},
     {'name': 'Richard Overy',
      'url': 'https://en.wikipedia.org/wiki/Richard_Overy'},
     {'name': 'Steven Ozment',
      'url': 'https://en.wikipedia.org/wiki/Steven_Ozment'},
     {'name': 'Thomas Pakenham',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Pakenham_(historian)'},
     {'name': 'Madhavan K. Palat',
      'url': 'https://en.wikipedia.org/wiki/Madhavan_K._Palat'},
     {'name': u'Hasan B\xfclent Paksoy',
      'url': 'https://en.wikipedia.org/wiki/Hasan_B%C3%BClent_Paksoy'},
     {'name': u'Ilan Papp\xe9',
      'url': 'https://en.wikipedia.org/wiki/Ilan_Papp%C3%A9'},
     {'name': 'Simo Parpola', 'url': 'https://en.wikipedia.org/wiki/Simo_Parpola'},
     {'name': 'J. H. Parry', 'url': 'https://en.wikipedia.org/wiki/J._H._Parry'},
     {'name': 'T. T. Paterson',
      'url': 'https://en.wikipedia.org/wiki/T._T._Paterson'},
     {'name': 'Fred Patten', 'url': 'https://en.wikipedia.org/wiki/Fred_Patten'},
     {'name': 'Peter Paret', 'url': 'https://en.wikipedia.org/wiki/Peter_Paret'},
     {'name': 'Geoffrey Parker',
      'url': 'https://en.wikipedia.org/wiki/Geoffrey_Parker_(historian)'},
     {'name': 'Stanley G. Payne',
      'url': 'https://en.wikipedia.org/wiki/Stanley_G._Payne'},
     {'name': 'Abel Paz', 'url': 'https://en.wikipedia.org/wiki/Abel_Paz'},
     {'name': 'Morgan D. Peoples',
      'url': 'https://en.wikipedia.org/wiki/Morgan_D._Peoples'},
     {'name': 'William Armstrong Percy',
      'url': 'https://en.wikipedia.org/wiki/William_Armstrong_Percy'},
     {'name': 'Bradford Perkins',
      'url': 'https://en.wikipedia.org/wiki/Bradford_Perkins_(historian)'},
     {'name': 'Detlev Peukert',
      'url': 'https://en.wikipedia.org/wiki/Detlev_Peukert'},
     {'name': 'Liza Picard', 'url': 'https://en.wikipedia.org/wiki/Liza_Picard'},
     {'name': 'David Pietrusza',
      'url': 'https://en.wikipedia.org/wiki/David_Pietrusza'},
     {'name': 'Boris B. Piotrovsky',
      'url': 'https://en.wikipedia.org/wiki/Boris_B._Piotrovsky'},
     {'name': 'Richard Pipes',
      'url': 'https://en.wikipedia.org/wiki/Richard_Pipes'},
     {'name': 'J.H. Plumb', 'url': 'https://en.wikipedia.org/wiki/J.H._Plumb'},
     {'name': 'J. G. A. Pocock',
      'url': 'https://en.wikipedia.org/wiki/J._G._A._Pocock'},
     {'name': 'Kwok Kin Poon',
      'url': 'https://en.wikipedia.org/wiki/Kwok_Kin_Poon'},
     {'name': 'Barbara Corrado Pope',
      'url': 'https://en.wikipedia.org/wiki/Barbara_Corrado_Pope'},
     {'name': 'Roy Porter', 'url': 'https://en.wikipedia.org/wiki/Roy_Porter'},
     {'name': 'Norman Pounds',
      'url': 'https://en.wikipedia.org/wiki/Norman_Pounds'},
     {'name': 'Gordon W. Prange',
      'url': 'https://en.wikipedia.org/wiki/Gordon_W._Prange'},
     {'name': 'Joshua Prawer',
      'url': 'https://en.wikipedia.org/wiki/Joshua_Prawer'},
     {'name': 'Michael Prestwich',
      'url': 'https://en.wikipedia.org/wiki/Michael_Prestwich'},
     {'name': 'Clement Alexander Price',
      'url': 'https://en.wikipedia.org/wiki/Clement_Alexander_Price'},
     {'name': 'Francis Paul Prucha',
      'url': 'https://en.wikipedia.org/wiki/Francis_Paul_Prucha'},
     {'name': 'Janko Prunk', 'url': 'https://en.wikipedia.org/wiki/Janko_Prunk'},
     {'name': 'Carroll Quigley',
      'url': 'https://en.wikipedia.org/wiki/Carroll_Quigley'},
     {'name': 'Marc Raeff', 'url': 'https://en.wikipedia.org/wiki/Marc_Raeff'},
     {'name': 'Werner Rahn', 'url': 'https://en.wikipedia.org/wiki/Werner_Rahn'},
     {'name': 'Jack N. Rakove',
      'url': 'https://en.wikipedia.org/wiki/Jack_N._Rakove'},
     {'name': u'\u0160erbo Rastoder',
      'url': 'https://en.wikipedia.org/wiki/%C5%A0erbo_Rastoder'},
     {'name': u'Ren\xe9 R\xe9mond',
      'url': 'https://en.wikipedia.org/wiki/Ren%C3%A9_R%C3%A9mond'},
     {'name': 'Timothy Reuter',
      'url': 'https://en.wikipedia.org/wiki/Timothy_Reuter'},
     {'name': 'Henry A. Reynolds',
      'url': 'https://en.wikipedia.org/wiki/Henry_A._Reynolds'},
     {'name': 'Susan Reynolds',
      'url': 'https://en.wikipedia.org/wiki/Susan_Reynolds'},
     {'name': 'Richard Rhodes',
      'url': 'https://en.wikipedia.org/wiki/Richard_Rhodes'},
     {'name': 'Nicholas V. Riasanovsky',
      'url': 'https://en.wikipedia.org/wiki/Nicholas_V._Riasanovsky'},
     {'name': 'Admiral Sir Herbert Richmond',
      'url': 'https://en.wikipedia.org/wiki/Herbert_Richmond'},
     {'name': 'Jonathan Riley-Smith',
      'url': 'https://en.wikipedia.org/wiki/Jonathan_Riley-Smith'},
     {'name': 'Blaze Ristovski',
      'url': 'https://en.wikipedia.org/wiki/Blaze_Ristovski'},
     {'name': 'Charles Ritcheson',
      'url': 'https://en.wikipedia.org/wiki/Charles_Ritcheson'},
     {'name': 'Gerhard Ritter',
      'url': 'https://en.wikipedia.org/wiki/Gerhard_Ritter'},
     {'name': 'Andrew Roberts',
      'url': 'https://en.wikipedia.org/wiki/Andrew_Roberts_(historian)'},
     {'name': 'J. M. Roberts',
      'url': 'https://en.wikipedia.org/wiki/J._M._Roberts'},
     {'name': 'N.A.M. Rodger',
      'url': 'https://en.wikipedia.org/wiki/N.A.M._Rodger'},
     {'name': 'Walter Rodney',
      'url': 'https://en.wikipedia.org/wiki/Walter_Rodney'},
     {'name': 'William Ledyard Rodgers',
      'url': 'https://en.wikipedia.org/wiki/William_Ledyard_Rodgers'},
     {'name': 'Theodore Ropp',
      'url': 'https://en.wikipedia.org/wiki/Theodore_Ropp'},
     {'name': 'W.J. Rorabaugh',
      'url': 'https://en.wikipedia.org/wiki/W.J._Rorabaugh'},
     {'name': 'Ron Rosenbaum',
      'url': 'https://en.wikipedia.org/wiki/Ron_Rosenbaum'},
     {'name': 'Charles E. Rosenberg',
      'url': 'https://en.wikipedia.org/wiki/Charles_E._Rosenberg'},
     {'name': 'Stephen Roskill',
      'url': 'https://en.wikipedia.org/wiki/Stephen_Roskill'},
     {'name': 'Maarten van Rossem',
      'url': 'https://en.wikipedia.org/wiki/Maarten_van_Rossem'},
     {'name': u'Mar\xeda Rostworowski',
      'url': 'https://en.wikipedia.org/wiki/Mar%C3%ADa_Rostworowski'},
     {'name': 'Theodore Roosevelt',
      'url': 'https://en.wikipedia.org/wiki/Theodore_Roosevelt'},
     {'name': 'Michael Rostovtzeff',
      'url': 'https://en.wikipedia.org/wiki/Michael_Rostovtzeff'},
     {'name': 'Hans Rothfels',
      'url': 'https://en.wikipedia.org/wiki/Hans_Rothfels'},
     {'name': 'Sheila Rowbotham',
      'url': 'https://en.wikipedia.org/wiki/Sheila_Rowbotham'},
     {'name': 'Herbert H. Rowen',
      'url': 'https://en.wikipedia.org/wiki/Herbert_H._Rowen'},
     {'name': 'A. L. Rowse', 'url': 'https://en.wikipedia.org/wiki/A._L._Rowse'},
     {'name': 'Miri Rubin', 'url': 'https://en.wikipedia.org/wiki/Miri_Rubin'},
     {'name': u'George Rud\xe9',
      'url': 'https://en.wikipedia.org/wiki/George_Rud%C3%A9'},
     {'name': 'R. J. Rummel', 'url': 'https://en.wikipedia.org/wiki/R._J._Rummel'},
     {'name': 'Steven Runciman',
      'url': 'https://en.wikipedia.org/wiki/Steven_Runciman'},
     {'name': 'Leila J.Rupp', 'url': 'https://en.wikipedia.org/wiki/Leila_J.Rupp'},
     {'name': 'Conrad Russell',
      'url': 'https://en.wikipedia.org/wiki/Conrad_Russell'},
     {'name': 'Cornelius Ryan',
      'url': 'https://en.wikipedia.org/wiki/Cornelius_Ryan'},
     {'name': 'Boris Rybakov',
      'url': 'https://en.wikipedia.org/wiki/Boris_Rybakov'},
     {'name': 'Edgar V. Saks',
      'url': 'https://en.wikipedia.org/wiki/Edgar_V._Saks'},
     {'name': 'Richard G. Salomon',
      'url': 'https://en.wikipedia.org/wiki/Richard_G._Salomon'},
     {'name': 'S. Srikanta Sastri',
      'url': 'https://en.wikipedia.org/wiki/S._Srikanta_Sastri'},
     {'name': 'J. Salwyn Schapiro',
      'url': 'https://en.wikipedia.org/wiki/J._Salwyn_Schapiro'},
     {'name': 'Dominic Sandbrook',
      'url': 'https://en.wikipedia.org/wiki/Dominic_Sandbrook'},
     {'name': 'Usha Sanyal', 'url': 'https://en.wikipedia.org/wiki/Usha_Sanyal'},
     {'name': 'Simon Schama', 'url': 'https://en.wikipedia.org/wiki/Simon_Schama'},
     ...]



At this point it's time to start loading individual pages, and loading content. Here I explore with what xpath work best to extract the type of information I want to extract. 

To do gender at this moment, I'm counting how often the words 'he' and 'she' come up, as wikipedia biographies infrequently refer to their subject by name after the few few sentences, and instead use the gendered pronoun to call back. If there are more ' he ' than ' she ', I presume the article is male and vice-versa. This is isn't perfect, but we're just doing a quick and dirty investigation.

I also need to extract some sort of date to the article. Unfortunately not all historians have exhaustive articles and there are structural differences to the html, based on how high profile the article is. I turned to regex to pull the year from the the bio, regex is a text matching language, allowing you to define a pattern and return all matches. In this case I'm looking for 2,3,4 or for digit number, and I'm looking for the first time it's said. Wikipedia generally structures it's articles (even stubs) "Sophie XXX (c. 1991)" so we're depending on that logic, and our knowledge of wikipedia. You can see how scraped data can get messy quickly, with edge cases mucking up the data. 


```python
import re
```


```python
req_historian = requests.get(historians[7]['url'])
#print req_historian.text
print historians[6]['url'] # easy clicking to check
historian_page = html.fromstring(req_historian.text)
text = historian_page.xpath('//div[@id="mw-content-text"]/p//text()')
text = "".join(text)
text.replace("\u",'') #clean up
text.replace("\n",'') 
print
print text
print "'he'  counts:", text.count(' he '), text.count(' He ')
print "'she' counts:",text.count(' she '), text.count(' She ')


#test this before putting it online
year = re.compile("\d{3,4}")#this regex grabs years
    
hist_year = year.findall(text)[0]
hist_year = hist_year
print hist_year
```

    https://en.wikipedia.org/wiki/Berossus
    
    Ptolemy I Soter I (Ancient Greek: Πτολεμαῖος Σωτήρ, Ptolemaĩos Sōtḗr, i.e. Ptolemy (pronounced /ˈtɒləmi/) the Savior), also known as Ptolemy Lagides[1] (c. 367 BC – 283/2 BC), was a Macedonian Greek[2][3][4][5][6] general under Alexander the Great, one of the three Diadochi who succeeded to his empire. Ptolemy became ruler of Egypt (323–283/2 BC) and founded a dynasty which ruled it for the next three centuries, turning Egypt into a Hellenistic kingdom and Alexandria into a center of Greek culture. He assimilated some aspects of Egyptian culture, however, assuming the traditional title pharaoh in 305/4 BC. The use of the title of pharaoh was often situational: pharaoh was used for an Egyptian audience, and Basileus for a Greek audience, as exemplified by Egyptian coinage.Like all Macedonian nobles, Ptolemy I Soter claimed descent from Heracles, the mythical founder of the Argead dynasty that ruled Macedon. Ptolemy's mother was Arsinoe of Macedon, and, while his father is unknown, ancient sources variously describe him either as the son of Lagus, a Macedonian nobleman, or as an illegitimate son of Philip II of Macedon (which, if true, would have made Ptolemy the half-brother of Alexander), but it is possible that this is a later myth fabricated to glorify the Ptolemaic dynasty. Ptolemy was one of Alexander's most trusted generals, and was among the seven somatophylakes (bodyguards) attached to his person. He was a few years older than Alexander and had been his intimate friend since childhood.He was succeeded by his son Ptolemy II Philadelphus.Ptolemy served with Alexander from his first campaigns, and played a principal part in the later campaigns in Afghanistan and India. He participated in the Battle of Issus commanding troops on the left wing under the authority of Parmenion, later he accompanied Alexander during his journey to the Oracle in the Siwa Oasis where he was proclaimed a son of Zeus.[7] Ptolemy had his first independent command during the campaign against the rebel Bessus whom Ptolemy captured and handed over to Alexander for execution.[8] During Alexander's campaign in the Indian subcontinent Ptolemy was in command of the advance guard at the siege of Aornos and fought at the Battle of the Hydaspes River.When Alexander died in 323 BC, Ptolemy is said to have instigated the resettlement of the empire made at Babylon. Through the Partition of Babylon, he was appointed satrap of Egypt, under the nominal kings Philip III Arrhidaeus and the infant Alexander IV; the former satrap, the Greek Cleomenes, stayed on as his deputy. Ptolemy quickly moved, without authorization, to subjugate Cyrenaica.By custom, kings in Macedonia asserted their right to the throne by burying their predecessor. Probably because he wanted to pre-empt Perdiccas, the imperial regent, from staking his claim in this way, Ptolemy took great pains in acquiring the body of Alexander the Great, placing it temporarily in Memphis, Egypt. Ptolemy then openly joined the coalition against Perdiccas.[9]Perdiccas appears to have suspected Ptolemy of aiming for the throne himself, and may have decided that Ptolemy was his most dangerous rival. Ptolemy executed Cleomenes for spying on behalf of Perdiccas — this removed the chief check on his authority, and allowed Ptolemy to obtain the huge sum that Cleomenes had accumulated.[9]In 321 BC, Perdiccas attempted to invade Egypt only to fall at the hands of his own men.[10] Ptolemy's decision to defend the Nile against Perdiccas's attempt to force it ended in fiasco for Perdiccas, with the loss of 2000 men. This failure was a fatal blow to Perdiccas' reputation, and he was murdered in his tent by two of his subordinates. Ptolemy immediately crossed the Nile, to provide supplies to what had the day before been an enemy army. Ptolemy was offered the regency in place of Perdiccas; but he declined.[11] Ptolemy was consistent in his policy of securing a power base, while never succumbing to the temptation of risking all to succeed Alexander.[12]In the long wars that followed between the different Diadochi, Ptolemy's first goal was to hold Egypt securely, and his second was to secure control in the outlying areas: Cyrenaica and Cyprus, as well as Syria, including the province of Judea. His first occupation of Syria was in 318, and he established at the same time a protectorate over the petty kings of Cyprus. When Antigonus One-Eye, master of Asia in 315, showed dangerous ambitions, Ptolemy joined the coalition against him, and on the outbreak of war, evacuated Syria. In Cyprus, he fought the partisans of Antigonus, and re-conquered the island (313). A revolt in Cyrene was crushed the same year.In 312, Ptolemy and Seleucus, the fugitive satrap of Babylonia, both invaded Syria, and defeated Demetrius Poliorcetes ("besieger of cities"), the son of Antigonus, in the Battle of Gaza. Again he occupied Syria, and again—after only a few months, when Demetrius had won a battle over his general, and Antigonus entered Syria in force—he evacuated it. In 311, a peace was concluded between the combatants. Soon after this, the surviving 13-year-old king, Alexander IV, was murdered in Macedonia on the orders of Cassander, leaving the satrap of Egypt absolutely his own master.The peace did not last long, and in 309 Ptolemy personally commanded a fleet that detached the coastal towns of Lycia and Caria from Antigonus, then crossed into Greece, where he took possession of Corinth, Sicyon and Megara (308 BC). In 306, a great fleet under Demetrius attacked Cyprus, and Ptolemy's brother Menelaus was defeated and captured in another decisive Battle of Salamis. Ptolemy's complete loss of Cyprus followed.The satraps Antigonus and Demetrius now each assumed the title of king; Ptolemy, as well as Cassander, Lysimachus and Seleucus I Nicator, responded by doing the same. In the winter of 306 BC, Antigonus tried to follow up his victory in Cyprus by invading Egypt; but Ptolemy was strongest there, and successfully held the frontier against him. Ptolemy led no further overseas expeditions against Antigonus. However, he did send great assistance to Rhodes when it was besieged by Demetrius (305/304). Pausanias reports that the grateful Rhodians bestowed the name Soter ("saviour") upon him as a result of lifting the siege. This account is generally accepted by modern scholars, although the earliest datable mention of it is from coins issued by Ptolemy II in 263 BC.When the coalition against Antigonus was renewed in 302, Ptolemy joined it, and invaded Syria a third time, while Antigonus was engaged with Lysimachus in Asia Minor. On hearing a report that Antigonus had won a decisive victory there, he once again evacuated Syria. But when the news came that Antigonus had been defeated and slain by Lysimachus and Seleucus at the Battle of Ipsus in 301, he occupied Syria a fourth time.The other members of the coalition had assigned all Syria to Seleucus, after what they regarded as Ptolemy's desertion, and for the next hundred years, the question of the ownership of southern Syria (i.e., Judea) produced recurring warfare between the Seleucid and Ptolemaic dynasties. Henceforth, Ptolemy seems to have mingled as little as possible in the rivalries between Asia Minor and Greece; he lost what he held in Greece, but reconquered Cyprus in 295/294. Cyrene, after a series of rebellions, was finally subjugated about 300 and placed under his stepson Magas.In 289, Ptolemy made his son by Berenice—Ptolemy II Philadelphus—his co-regent. His eldest (legitimate) son, Ptolemy Keraunos, whose mother, Eurydice, the daughter of Antipater, had been repudiated, fled to the court of Lysimachus. Ptolemy also had a consort in Thaïs, the Athenian hetaera and one of Alexander's companions in his conquest of the ancient world. Ptolemy I Soter died in winter 283 or spring 282 at the age of 84.[13] Shrewd and cautious, he had a compact and well-ordered realm to show at the end of forty years of war. His reputation for bonhomie and liberality attached the floating soldier-class of Macedonians and other Greeks to his service, and was not insignificant; nor did he wholly neglect conciliation of the natives. He was a ready patron of letters, founding the Great Library of Alexandria.[14]Ptolemy himself wrote an eye-witness history of Alexander's campaigns (now lost).[15] In the second century AD, Ptolemy's history was used by Arrian of Nicomedia as one of his two main primary sources (alongside the history of Aristobulus of Cassandreia) for his own extant Anabasis of Alexander, and hence large parts of Ptolemy's history can be assumed to survive in paraphrase or précis in Arrian's work.[16] Arrian cites Ptolemy by name on only a few occasions, but it is likely enough that large stretches of Arrian's Anabasis ultimately reflect Ptolemy's version of events. Arrian once names Ptolemy as the author "whom I chiefly follow" (Anabasis 6.2.4), and in his Preface claims that Ptolemy seemed to him to be a particularly trustworthy source, "not only because he was present with Alexander on campaign, but also because he was himself a king, and hence lying would be more dishonourable for him than for anyone else" (Anabasis, Prologue).Ptolemy's lost history was long considered an objective work, distinguished by its straightforward honesty and sobriety, but more recent work has called this assessment into question. R. M. Errington argued that Ptolemy's history was characterised by persistent bias and self-aggrandisement, and by systematic blackening of the reputation of Perdiccas, one of Ptolemy's chief dynastic rivals after Alexander's death.[17] For example, Arrian's account of the fall of Thebes in 335 BC (Anabasis 1.8.1-1.8.8, a rare section of narrative explicitly attributed to Ptolemy by Arrian) shows several significant variations from the parallel account preserved in Diodorus Siculus (17.11-12), most notably in attributing a distinctly unheroic role in proceedings to Perdiccas. More recently, J. Roisman has argued that the case for Ptolemy's blackening of Perdiccas and others has been much exaggerated.[18]Ptolemy personally sponsored the great mathematician Euclid, but found Euclid's seminal work, the Elements, too difficult to study, so he asked if there were an easier way to master it. According to Proclus Euclid famously quipped: "Sire, there is no Royal Road to geometry."[19] Media related to Ptolemy I at Wikimedia Commons
    'he'  counts: 20 4
    'she' counts: 0 0
    367


Note, I went through many of these to make sure this assumption passed at least the common sense challenge. Now we've got almost everything down, it's just down to parsing it and saving the data! 


```python
len(historians)
```




    1158



I'm going to loop through and apply everything I figured out above to everything else, storing it in a dictionary objected that makes it easy to cast this to pandas and save it out. 


```python
for i in historians:
    req_historian = requests.get(i['url'])
    historian_page = html.fromstring(req_historian.text)
    text = historian_page.xpath('//div[@id="mw-content-text"]/p//text()')
    text = "".join(text)
    text.replace("\u",'')
    text.replace("\n",'')
    #print
   
    he = text.count(' he ') + text.count(' He ')
    she= text.count(' she ') + text.count(' She ')
    
    year = re.compile("\d{3,4}")#this regex grabs years
    
    hist_year = year.search(text)
    try: 
        hist_year = hist_year.group()
    except AttributeError: 
        pass
        
    
    gender = ""
    if he > she: 
        gender = "Male"
    if she > he: 
        gender = "Female"
        #print i['url']
    
    i['gender'] = gender
    try:
        i['birth_year'] = hist_year
    except:
        i['birth_year'] = None
```


```python
historians #all done! 
```




    [{'birth_year': u'484',
      'gender': 'Male',
      'name': 'Herodotus',
      'url': 'https://en.wikipedia.org/wiki/Herodotus'},
     {'birth_year': u'460',
      'gender': 'Male',
      'name': 'Thucydides',
      'url': 'https://en.wikipedia.org/wiki/Thucydides'},
     {'birth_year': u'430',
      'gender': 'Male',
      'name': 'Xenophon',
      'url': 'https://en.wikipedia.org/wiki/Xenophon'},
     {'birth_year': u'401',
      'gender': 'Male',
      'name': 'Ctesias',
      'url': 'https://en.wikipedia.org/wiki/Ctesias'},
     {'birth_year': u'380',
      'gender': 'Male',
      'name': 'Theopompus',
      'url': 'https://en.wikipedia.org/wiki/Theopompus'},
     {'birth_year': u'370',
      'gender': 'Male',
      'name': 'Eudemus of Rhodes',
      'url': 'https://en.wikipedia.org/wiki/Eudemus_of_Rhodes'},
     {'birth_year': u'290',
      'gender': 'Male',
      'name': 'Berossus',
      'url': 'https://en.wikipedia.org/wiki/Berossus'},
     {'birth_year': u'367',
      'gender': 'Male',
      'name': 'Ptolemy I Soter',
      'url': 'https://en.wikipedia.org/wiki/Ptolemy_I_Soter'},
     {'birth_year': u'350',
      'gender': 'Male',
      'name': 'Duris of Samos',
      'url': 'https://en.wikipedia.org/wiki/Duris_of_Samos'},
     {'birth_year': u'323',
      'gender': 'Male',
      'name': 'Manetho',
      'url': 'https://en.wikipedia.org/wiki/Manetho'},
     {'birth_year': u'345',
      'gender': 'Male',
      'name': 'Timaeus of Tauromenium',
      'url': 'https://en.wikipedia.org/wiki/Timaeus_(historian)'},
     {'birth_year': '200',
      'gender': 'Male',
      'name': 'Quintus Fabius Pictor',
      'url': 'https://en.wikipedia.org/wiki/Quintus_Fabius_Pictor'},
     {'birth_year': u'250',
      'gender': 'Male',
      'name': 'Artapanus of Alexandria',
      'url': 'https://en.wikipedia.org/wiki/Artapanus_of_Alexandria'},
     {'birth_year': u'234',
      'gender': 'Male',
      'name': 'Cato the Elder',
      'url': 'https://en.wikipedia.org/wiki/Cato_the_Elder'},
     {'birth_year': '155',
      'gender': 'Male',
      'name': 'Gaius Acilius',
      'url': 'https://en.wikipedia.org/wiki/Gaius_Acilius'},
     {'birth_year': u'200',
      'gender': 'Male',
      'name': 'Polybius',
      'url': 'https://en.wikipedia.org/wiki/Polybius'},
     {'birth_year': '158',
      'gender': 'Male',
      'name': 'Sempronius Asellio',
      'url': 'https://en.wikipedia.org/wiki/Sempronius_Asellio'},
     {'birth_year': u'165',
      'gender': 'Male',
      'name': 'Sima Tan',
      'url': 'https://en.wikipedia.org/wiki/Sima_Tan'},
     {'birth_year': u'145',
      'gender': 'Male',
      'name': 'Sima Qian',
      'url': 'https://en.wikipedia.org/wiki/Sima_Qian'},
     {'birth_year': u'169',
      'gender': 'Male',
      'name': 'Agatharchides',
      'url': 'https://en.wikipedia.org/wiki/Agatharchides'},
     {'birth_year': u'135',
      'gender': 'Male',
      'name': 'Posidonius',
      'url': 'https://en.wikipedia.org/wiki/Posidonius'},
     {'birth_year': u'100',
      'gender': 'Male',
      'name': 'Julius Caesar',
      'url': 'https://en.wikipedia.org/wiki/Julius_Caesar'},
     {'birth_year': u'1968',
      'gender': 'Male',
      'name': 'Diodorus of Sicily',
      'url': 'https://en.wikipedia.org/wiki/Diodorus_Siculus'},
     {'birth_year': u'111',
      'gender': 'Male',
      'name': 'Sallust',
      'url': 'https://en.wikipedia.org/wiki/Sallust'},
     {'birth_year': None,
      'gender': 'Male',
      'name': 'Liu Xiang (scholar)',
      'url': 'https://en.wikipedia.org/wiki/Liu_Xiang_(scholar)'},
     {'birth_year': u'1870',
      'gender': 'Male',
      'name': 'Theophanes of Mytilene',
      'url': 'https://en.wikipedia.org/wiki/Theophanes_of_Mytilene'},
     {'birth_year': u'300',
      'gender': 'Male',
      'name': 'Dionysius of Halicarnassus',
      'url': 'https://en.wikipedia.org/wiki/Dionysius_of_Halicarnassus'},
     {'birth_year': u'1469',
      'gender': 'Male',
      'name': 'Strabo',
      'url': 'https://en.wikipedia.org/wiki/Strabo'},
     {'birth_year': u'753',
      'gender': 'Male',
      'name': 'Livy',
      'url': 'https://en.wikipedia.org/wiki/Livy'},
     {'birth_year': u'311',
      'gender': 'Male',
      'name': 'Marcus Velleius Paterculus',
      'url': 'https://en.wikipedia.org/wiki/Marcus_Velleius_Paterculus'},
     {'birth_year': u'1557',
      'gender': 'Male',
      'name': 'Memnon of Heraclea',
      'url': 'https://en.wikipedia.org/wiki/Memnon_of_Heraclea'},
     {'birth_year': None,
      'gender': 'Male',
      'name': 'Ban Biao',
      'url': 'https://en.wikipedia.org/wiki/Ban_Biao'},
     {'birth_year': u'167',
      'gender': 'Male',
      'name': 'Quintus Curtius Rufus',
      'url': 'https://en.wikipedia.org/wiki/Quintus_Curtius_Rufus'},
     {'birth_year': None,
      'gender': 'Male',
      'name': 'Ban Gu',
      'url': 'https://en.wikipedia.org/wiki/Ban_Gu'},
     {'birth_year': u'100',
      'gender': 'Male',
      'name': 'Flavius Josephus',
      'url': 'https://en.wikipedia.org/wiki/Josephus'},
     {'birth_year': None,
      'gender': 'Female',
      'name': 'Pamphile of Epidaurus',
      'url': 'https://en.wikipedia.org/wiki/Pamphile_of_Epidaurus'},
     {'birth_year': u'116',
      'gender': 'Female',
      'name': 'Ban Zhao',
      'url': 'https://en.wikipedia.org/wiki/Ban_Zhao'},
     {'birth_year': u'167',
      'gender': 'Male',
      'name': 'Thallus',
      'url': 'https://en.wikipedia.org/wiki/Thallus_(historian)'},
     {'birth_year': u'120',
      'gender': 'Male',
      'name': 'Plutarch',
      'url': 'https://en.wikipedia.org/wiki/Plutarch'},
     {'birth_year': u'120',
      'gender': 'Male',
      'name': 'Gaius Cornelius Tacitus',
      'url': 'https://en.wikipedia.org/wiki/Gaius_Cornelius_Tacitus'},
     {'birth_year': u'121',
      'gender': 'Male',
      'name': 'Suetonius',
      'url': 'https://en.wikipedia.org/wiki/Lives_of_the_Twelve_Caesars'},
     {'birth_year': u'165',
      'gender': 'Male',
      'name': 'Appian',
      'url': 'https://en.wikipedia.org/wiki/Appian'},
     {'birth_year': u'146',
      'gender': 'Male',
      'name': 'Arrian',
      'url': 'https://en.wikipedia.org/wiki/Arrian'},
     {'birth_year': u'376',
      'gender': 'Male',
      'name': 'Lucius Ampelius',
      'url': 'https://en.wikipedia.org/wiki/Liber_Memorialis'},
     {'birth_year': u'155',
      'gender': 'Male',
      'name': 'Dio Cassius',
      'url': 'https://en.wikipedia.org/wiki/Dio_Cassius'},
     {'birth_year': u'170',
      'gender': 'Male',
      'name': 'Herodian',
      'url': 'https://en.wikipedia.org/wiki/Herodian'},
     {'birth_year': u'160',
      'gender': 'Male',
      'name': 'Sextus Julius Africanus',
      'url': 'https://en.wikipedia.org/wiki/Sextus_Julius_Africanus'},
     {'birth_year': u'200',
      'gender': 'Male',
      'name': u'Diogenes La\xebrtius',
      'url': 'https://en.wikipedia.org/wiki/Diogenes_La%C3%ABrtius'},
     {'birth_year': u'233',
      'gender': 'Male',
      'name': 'Chen Shou',
      'url': 'https://en.wikipedia.org/wiki/Chen_Shou'},
     {'birth_year': u'260',
      'gender': 'Male',
      'name': 'Eusebius of Caesarea',
      'url': 'https://en.wikipedia.org/wiki/Eusebius_of_Caesarea'},
     {'birth_year': u'325',
      'gender': 'Male',
      'name': 'Ammianus Marcellinus',
      'url': 'https://en.wikipedia.org/wiki/Ammianus_Marcellinus'},
     {'birth_year': u'337',
      'gender': 'Male',
      'name': 'Fa-Hien',
      'url': 'https://en.wikipedia.org/wiki/Fa-Hien'},
     {'birth_year': u'340',
      'gender': 'Male',
      'name': 'Rufinus of Aquileia',
      'url': 'https://en.wikipedia.org/wiki/Rufinus_of_Aquileia'},
     {'birth_year': u'368',
      'gender': 'Male',
      'name': 'Philostorgius',
      'url': 'https://en.wikipedia.org/wiki/Philostorgius'},
     {'birth_year': u'380',
      'gender': 'Male',
      'name': 'Socrates of Constantinople',
      'url': 'https://en.wikipedia.org/wiki/Socrates_of_Constantinople'},
     {'birth_year': u'393',
      'gender': 'Male',
      'name': 'Theodoret',
      'url': 'https://en.wikipedia.org/wiki/Theodoret'},
     {'birth_year': u'398',
      'gender': 'Male',
      'name': 'Fan Ye (historian)',
      'url': 'https://en.wikipedia.org/wiki/Fan_Ye_(historian)'},
     {'birth_year': u'410',
      'gender': 'Male',
      'name': 'Priscus',
      'url': 'https://en.wikipedia.org/wiki/Priscus'},
     {'birth_year': u'400',
      'gender': 'Male',
      'name': 'Sozomen',
      'url': 'https://en.wikipedia.org/wiki/Sozomen'},
     {'birth_year': u'400',
      'gender': 'Male',
      'name': 'Salvian',
      'url': 'https://en.wikipedia.org/wiki/Salvian'},
     {'birth_year': u'410',
      'gender': 'Male',
      'name': 'Movses Khorenatsi',
      'url': 'https://en.wikipedia.org/wiki/Movses_Khorenatsi'},
     {'birth_year': u'441',
      'gender': 'Male',
      'name': 'Shen Yue',
      'url': 'https://en.wikipedia.org/wiki/Shen_Yue'},
     {'birth_year': u'491',
      'gender': 'Male',
      'name': 'John Malalas',
      'url': 'https://en.wikipedia.org/wiki/John_Malalas'},
     {'birth_year': u'490',
      'gender': 'Male',
      'name': 'Zosimus',
      'url': 'https://en.wikipedia.org/wiki/Zosimus'},
     {'birth_year': u'500',
      'gender': 'Male',
      'name': 'Procopius',
      'url': 'https://en.wikipedia.org/wiki/Procopius'},
     {'birth_year': u'551',
      'gender': 'Male',
      'name': 'Jordanes',
      'url': 'https://en.wikipedia.org/wiki/Jordanes'},
     {'birth_year': u'538',
      'gender': 'Male',
      'name': 'Gregory of Tours',
      'url': 'https://en.wikipedia.org/wiki/Gregory_of_Tours'},
     {'birth_year': u'580',
      'gender': 'Male',
      'name': 'Wei Zheng',
      'url': 'https://en.wikipedia.org/wiki/Wei_Zheng'},
     {'birth_year': '600',
      'gender': 'Female',
      'name': 'Baudovinia',
      'url': 'https://en.wikipedia.org/wiki/Baudovinia'},
     {'birth_year': u'637',
      'gender': 'Male',
      'name': 'Yao Silian',
      'url': 'https://en.wikipedia.org/wiki/Yao_Silian'},
     {'birth_year': u'579',
      'gender': 'Male',
      'name': 'Fang Xuanling',
      'url': 'https://en.wikipedia.org/wiki/Fang_Xuanling'},
     {'birth_year': u'624',
      'gender': 'Male',
      'name': 'Adamnan',
      'url': 'https://en.wikipedia.org/wiki/Adamnan'},
     {'birth_year': u'672',
      'gender': 'Male',
      'name': 'Bede',
      'url': 'https://en.wikipedia.org/wiki/Bede'},
     {'birth_year': u'657',
      'gender': '',
      'name': u'T\xedrech\xe1n',
      'url': 'https://en.wikipedia.org/wiki/T%C3%ADrech%C3%A1n'},
     {'birth_year': '650',
      'gender': 'Male',
      'name': 'Cogitosus',
      'url': 'https://en.wikipedia.org/wiki/Cogitosus'},
     {'birth_year': u'697',
      'gender': 'Male',
      'name': 'Muirchu moccu Machtheni',
      'url': 'https://en.wikipedia.org/wiki/Muirchu_moccu_Machtheni'},
     {'birth_year': u'720',
      'gender': 'Male',
      'name': 'Paul the Deacon',
      'url': 'https://en.wikipedia.org/wiki/Paul_the_Deacon'},
     {'birth_year': u'661',
      'gender': 'Male',
      'name': 'Liu Zhiji',
      'url': 'https://en.wikipedia.org/wiki/Liu_Zhiji'},
     {'birth_year': u'723',
      'gender': 'Male',
      'name': u'\u014c no Yasumaro',
      'url': 'https://en.wikipedia.org/wiki/%C5%8C_no_Yasumaro'},
     {'birth_year': u'885',
      'gender': 'Male',
      'name': 'Constantine of Preslav',
      'url': 'https://en.wikipedia.org/wiki/Constantine_of_Preslav'},
     {'birth_year': u'809',
      'gender': 'Male',
      'name': 'Nennius',
      'url': 'https://en.wikipedia.org/wiki/Nennius'},
     {'birth_year': u'819',
      'gender': 'Male',
      'name': 'Martianus Hiberniensis',
      'url': 'https://en.wikipedia.org/wiki/Martianus_Hiberniensis'},
     {'birth_year': u'775',
      'gender': 'Male',
      'name': 'Einhard',
      'url': 'https://en.wikipedia.org/wiki/Einhard'},
     {'birth_year': u'840',
      'gender': 'Male',
      'name': 'Notker of St Gall',
      'url': 'https://en.wikipedia.org/wiki/Notker_of_St_Gall'},
     {'birth_year': u'915',
      'gender': 'Male',
      'name': u'Regino of Pr\xfcm',
      'url': 'https://en.wikipedia.org/wiki/Regino_of_Pr%C3%BCm'},
     {'birth_year': u'909',
      'gender': 'Male',
      'name': 'Asser',
      'url': 'https://en.wikipedia.org/wiki/Asser'},
     {'birth_year': u'224',
      'gender': 'Male',
      'name': 'Muhammad al-Tabari',
      'url': 'https://en.wikipedia.org/wiki/Muhammad_ibn_Jarir_al-Tabari'},
     {'birth_year': u'888',
      'gender': 'Male',
      'name': 'Liu Xu',
      'url': 'https://en.wikipedia.org/wiki/Liu_Xu'},
     {'birth_year': u'920',
      'gender': 'Male',
      'name': 'Liutprand of Cremona',
      'url': 'https://en.wikipedia.org/wiki/Liutprand_of_Cremona'},
     {'birth_year': u'925',
      'gender': 'Male',
      'name': 'Li Fang',
      'url': 'https://en.wikipedia.org/wiki/Li_Fang_(Song_dynasty)'},
     {'birth_year': u'925',
      'gender': 'Male',
      'name': 'Heriger of Lobbes',
      'url': 'https://en.wikipedia.org/wiki/Heriger_of_Lobbes'},
     {'birth_year': u'973',
      'gender': 'Male',
      'name': 'Al-Biruni',
      'url': 'https://en.wikipedia.org/wiki/Al-Biruni'},
     {'birth_year': u'975',
      'gender': 'Male',
      'name': 'Thietmar of Merseburg',
      'url': 'https://en.wikipedia.org/wiki/Thietmar_of_Merseburg'},
     {'birth_year': None,
      'gender': 'Male',
      'name': 'Ibn Rustah',
      'url': 'https://en.wikipedia.org/wiki/Ibn_Rustah'},
     {'birth_year': u'998',
      'gender': 'Male',
      'name': 'Song Qi',
      'url': 'https://en.wikipedia.org/wiki/Song_Qi'},
     {'birth_year': u'1007',
      'gender': 'Male',
      'name': 'Ouyang Xiu',
      'url': 'https://en.wikipedia.org/wiki/Ouyang_Xiu'},
     {'birth_year': u'1100',
      'gender': 'Male',
      'name': 'Albert of Aix',
      'url': 'https://en.wikipedia.org/wiki/Albert_of_Aix'},
     {'birth_year': u'1056',
      'gender': 'Male',
      'name': 'Nestor the Chronicler',
      'url': 'https://en.wikipedia.org/wiki/Nestor_the_Chronicler'},
     {'birth_year': u'1115',
      'gender': 'Male',
      'name': 'Gallus Anonymus',
      'url': 'https://en.wikipedia.org/wiki/Gallus_Anonymus'},
     {'birth_year': u'1022',
      'gender': 'Male',
      'name': 'Michael Attaleiates',
      'url': 'https://en.wikipedia.org/wiki/Michael_Attaleiates'},
     {'birth_year': u'1017',
      'gender': 'Male',
      'name': 'Michael Psellus',
      'url': 'https://en.wikipedia.org/wiki/Michael_Psellus'},
     {'birth_year': u'1019',
      'gender': 'Male',
      'name': 'Sima Guang',
      'url': 'https://en.wikipedia.org/wiki/Sima_Guang'},
     {'birth_year': u'1028',
      'gender': 'Male',
      'name': 'Marianus Scotus',
      'url': 'https://en.wikipedia.org/wiki/Marianus_Scotus'},
     {'birth_year': u'1055',
      'gender': 'Male',
      'name': 'Guibert of Nogent',
      'url': 'https://en.wikipedia.org/wiki/Guibert_of_Nogent'},
     {'birth_year': u'1050',
      'gender': 'Male',
      'name': 'Adam of Bremen',
      'url': 'https://en.wikipedia.org/wiki/Adam_of_Bremen'},
     {'birth_year': '1127',
      'gender': 'Male',
      'name': 'Galbert of Bruges',
      'url': 'https://en.wikipedia.org/wiki/Galbert_of_Bruges'},
     {'birth_year': u'1118',
      'gender': 'Male',
      'name': 'Florence of Worcester',
      'url': 'https://en.wikipedia.org/wiki/Florence_of_Worcester'},
     {'birth_year': u'1060',
      'gender': 'Male',
      'name': 'Eadmer',
      'url': 'https://en.wikipedia.org/wiki/Eadmer'},
     {'birth_year': u'1075',
      'gender': 'Male',
      'name': 'Kim Bu-sik',
      'url': 'https://en.wikipedia.org/wiki/Kim_Bu-sik'},
     {'birth_year': u'1095',
      'gender': 'Male',
      'name': 'William of Malmesbury',
      'url': 'https://en.wikipedia.org/wiki/William_of_Malmesbury'},
     {'birth_year': u'1129',
      'gender': 'Male',
      'name': 'Symeon of Durham',
      'url': 'https://en.wikipedia.org/wiki/Symeon_of_Durham'},
     {'birth_year': u'1083',
      'gender': 'Female',
      'name': 'Anna Comnena',
      'url': 'https://en.wikipedia.org/wiki/Anna_Comnena'},
     {'birth_year': u'1095',
      'gender': 'Male',
      'name': 'Usamah ibn Munqidh',
      'url': 'https://en.wikipedia.org/wiki/Usamah_ibn_Munqidh'},
     {'birth_year': u'1095',
      'gender': 'Male',
      'name': 'Geoffrey of Monmouth',
      'url': 'https://en.wikipedia.org/wiki/Geoffrey_of_Monmouth'},
     {'birth_year': u'1120',
      'gender': 'Male',
      'name': 'Helmold of Bosau',
      'url': 'https://en.wikipedia.org/wiki/Helmold_of_Bosau'},
     {'birth_year': u'1130',
      'gender': 'Male',
      'name': 'William of Tyre',
      'url': 'https://en.wikipedia.org/wiki/William_of_Tyre'},
     {'birth_year': u'1143',
      'gender': 'Male',
      'name': 'Alured of Beverley',
      'url': 'https://en.wikipedia.org/wiki/Alured_of_Beverley'},
     {'birth_year': u'1136',
      'gender': 'Male',
      'name': 'William of Newburgh',
      'url': 'https://en.wikipedia.org/wiki/William_of_Newburgh'},
     {'birth_year': u'1140',
      'gender': 'Male',
      'name': 'Svend Aagesen',
      'url': 'https://en.wikipedia.org/wiki/Svend_Aagesen'},
     {'birth_year': u'1164',
      'gender': 'Male',
      'name': 'Mohammed al-Baydhaq',
      'url': 'https://en.wikipedia.org/wiki/Mohammed_al-Baydhaq'},
     {'birth_year': '1140',
      'gender': 'Male',
      'name': 'John of Worcester',
      'url': 'https://en.wikipedia.org/wiki/John_of_Worcester'},
     {'birth_year': u'1146',
      'gender': 'Male',
      'name': 'Giraldus Cambrensis',
      'url': 'https://en.wikipedia.org/wiki/Giraldus_Cambrensis'},
     {'birth_year': u'1161',
      'gender': 'Male',
      'name': 'Wincenty Kadlubek',
      'url': 'https://en.wikipedia.org/wiki/Wincenty_Kadlubek'},
     {'birth_year': u'1190',
      'gender': 'Male',
      'name': 'Ambroise',
      'url': 'https://en.wikipedia.org/wiki/Ambroise'},
     {'birth_year': u'1160',
      'gender': 'Male',
      'name': 'Geoffroi de Villehardouin',
      'url': 'https://en.wikipedia.org/wiki/Geoffroi_de_Villehardouin'},
     {'birth_year': '1148',
      'gender': 'Male',
      'name': 'Kalhana',
      'url': 'https://en.wikipedia.org/wiki/Kalhana'},
     {'birth_year': u'1150',
      'gender': 'Male',
      'name': 'Saxo Grammaticus',
      'url': 'https://en.wikipedia.org/wiki/Saxo_Grammaticus'},
     {'birth_year': u'1118',
      'gender': 'Male',
      'name': 'Joannes Zonaras',
      'url': 'https://en.wikipedia.org/wiki/Joannes_Zonaras'},
     {'birth_year': u'1155',
      'gender': 'Male',
      'name': 'Nicetas Choniates',
      'url': 'https://en.wikipedia.org/wiki/Nicetas_Choniates'},
     {'birth_year': u'1179',
      'gender': 'Male',
      'name': 'Snorri Sturluson',
      'url': 'https://en.wikipedia.org/wiki/Snorri_Sturluson'},
     {'birth_year': '1185',
      'gender': 'Male',
      'name': 'Abdelwahid al-Marrakushi',
      'url': 'https://en.wikipedia.org/wiki/Abdelwahid_al-Marrakushi'},
     {'birth_year': u'1226',
      'gender': 'Male',
      'name': 'Ata al-Mulk Juvayni',
      'url': 'https://en.wikipedia.org/wiki/Ata_al-Mulk_Juvayni'},
     {'birth_year': u'1239',
      'gender': '',
      'name': 'Ibn al-Khabbaza',
      'url': 'https://en.wikipedia.org/wiki/Ibn_al-Khabbaza'},
     {'birth_year': u'1200',
      'gender': 'Male',
      'name': 'Matthew Paris',
      'url': 'https://en.wikipedia.org/wiki/Matthew_Paris'},
     {'birth_year': u'1210',
      'gender': 'Male',
      'name': 'Domentijan',
      'url': 'https://en.wikipedia.org/wiki/Domentijan'},
     {'birth_year': u'1206',
      'gender': 'Male',
      'name': 'Il-yeon',
      'url': 'https://en.wikipedia.org/wiki/Il-yeon'},
     {'birth_year': u'1221',
      'gender': 'Male',
      'name': 'Salimbene di Adam',
      'url': 'https://en.wikipedia.org/wiki/Salimbene_di_Adam'},
     {'birth_year': '1298',
      'gender': 'Male',
      'name': 'Abdelaziz al-Malzuzi',
      'url': 'https://en.wikipedia.org/wiki/Abdelaziz_al-Malzuzi'},
     {'birth_year': '1230',
      'gender': 'Male',
      'name': 'Templar of Tyre',
      'url': 'https://en.wikipedia.org/wiki/Templar_of_Tyre'},
     {'birth_year': u'1230',
      'gender': 'Male',
      'name': u'L\xea V\u0103n H\u01b0u',
      'url': 'https://en.wikipedia.org/wiki/L%C3%AA_V%C4%83n_H%C6%B0u'},
     {'birth_year': '1233',
      'gender': 'Male',
      'name': 'Adam of Eynsham',
      'url': 'https://en.wikipedia.org/wiki/Adam_of_Eynsham'},
     {'birth_year': u'1224',
      'gender': 'Male',
      'name': 'Jean de Joinville',
      'url': 'https://en.wikipedia.org/wiki/Jean_de_Joinville'},
     {'birth_year': '1305',
      'gender': 'Male',
      'name': 'Piers Langtoft',
      'url': 'https://en.wikipedia.org/wiki/Piers_Langtoft'},
     {'birth_year': u'1247',
      'gender': 'Male',
      'name': 'Rashid-al-Din Hamadani',
      'url': 'https://en.wikipedia.org/wiki/Rashid-al-Din_Hamadani'},
     {'birth_year': u'1276',
      'gender': 'Male',
      'name': 'Giovanni Villani',
      'url': 'https://en.wikipedia.org/wiki/Giovanni_Villani'},
     {'birth_year': u'1312',
      'gender': '',
      'name': 'Ibn Idhari',
      'url': 'https://en.wikipedia.org/wiki/Ibn_Idhari'},
     {'birth_year': u'1310',
      'gender': 'Male',
      'name': 'Ibn Abi Zar',
      'url': 'https://en.wikipedia.org/wiki/Ibn_Abi_Zar'},
     {'birth_year': u'1299',
      'gender': '',
      'name': 'Abdullah Wassaf',
      'url': 'https://en.wikipedia.org/wiki/Wassaf'},
     {'birth_year': u'1310',
      'gender': '',
      'name': 'Song Lian',
      'url': 'https://en.wikipedia.org/wiki/Song_Lian'},
     {'birth_year': u'1314',
      'gender': 'Male',
      'name': "Toqto'a",
      'url': 'https://en.wikipedia.org/wiki/Toqto%27a_(Yuan_Dynasty)'},
     {'birth_year': u'1332',
      'gender': 'Male',
      'name': 'ibn Khaldun',
      'url': 'https://en.wikipedia.org/wiki/Ibn_Khaldun'},
     {'birth_year': u'1286',
      'gender': 'Male',
      'name': 'John Clyn',
      'url': 'https://en.wikipedia.org/wiki/John_Clyn'},
     {'birth_year': u'1336',
      'gender': 'Male',
      'name': 'Baldassarre Bonaiuti',
      'url': 'https://en.wikipedia.org/wiki/Baldassarre_Bonaiuti'},
     {'birth_year': u'1337',
      'gender': 'Male',
      'name': 'Jean Froissart',
      'url': 'https://en.wikipedia.org/wiki/Jean_Froissart'},
     {'birth_year': u'1345',
      'gender': 'Male',
      'name': 'Dietrich of Nieheim',
      'url': 'https://en.wikipedia.org/wiki/Dietrich_of_Nieheim'},
     {'birth_year': u'1360',
      'gender': 'Male',
      'name': 'John of Fordun',
      'url': 'https://en.wikipedia.org/wiki/John_of_Fordun'},
     {'birth_year': u'1387',
      'gender': '',
      'name': u'Ruaidhri \xd3 Cian\xe1in',
      'url': 'https://en.wikipedia.org/wiki/Ruaidhri_%C3%93_Cian%C3%A1in'},
     {'birth_year': u'1364',
      'gender': 'Female',
      'name': 'Christine de Pizan',
      'url': 'https://en.wikipedia.org/wiki/Christine_de_Pizan'},
     {'birth_year': u'1370',
      'gender': 'Male',
      'name': u'\xc1lvar Garc\xeda de Santa Mar\xeda',
      'url': 'https://en.wikipedia.org/wiki/%C3%81lvar_Garc%C3%ADa_de_Santa_Mar%C3%ADa'},
     {'birth_year': u'1372',
      'gender': 'Male',
      'name': u'Se\xe1n M\xf3r \xd3 Dubhag\xe1in',
      'url': 'https://en.wikipedia.org/wiki/Se%C3%A1n_M%C3%B3r_%C3%93_Dubhag%C3%A1in'},
     {'birth_year': u'1373',
      'gender': '',
      'name': u'Adhamh \xd3 Cian\xe1in',
      'url': 'https://en.wikipedia.org/wiki/Adhamh_%C3%93_Cian%C3%A1in'},
     {'birth_year': u'1324',
      'gender': 'Male',
      'name': 'Ismail ibn al-Ahmar',
      'url': 'https://en.wikipedia.org/wiki/Ismail_ibn_al-Ahmar'},
     {'birth_year': u'1390',
      'gender': 'Male',
      'name': u'Giolla \xcdosa M\xf3r Mac Fhirbhisigh',
      'url': 'https://en.wikipedia.org/wiki/Giolla_%C3%8Dosa_M%C3%B3r_Mac_Fhirbhisigh'},
     {'birth_year': u'1378',
      'gender': 'Male',
      'name': 'Zhu Quan',
      'url': 'https://en.wikipedia.org/wiki/Zhu_Quan'},
     {'birth_year': None,
      'gender': '',
      'name': 'John Capgrave',
      'url': 'https://en.wikipedia.org/wiki/John_Capgrave'},
     {'birth_year': u'1384',
      'gender': 'Male',
      'name': 'Alphonsus A Sancta Maria',
      'url': 'https://en.wikipedia.org/wiki/Alphonsus_A_Sancta_Maria'},
     {'birth_year': u'1415',
      'gender': 'Male',
      'name': u'Jan D\u0142ugosz',
      'url': 'https://en.wikipedia.org/wiki/Jan_D%C5%82ugosz'},
     {'birth_year': u'1454',
      'gender': 'Male',
      'name': 'Sharaf ad-Din Ali Yazdi',
      'url': 'https://en.wikipedia.org/wiki/Sharaf_ad-Din_Ali_Yazdi'},
     {'birth_year': u'1439',
      'gender': 'Male',
      'name': u'Cathal \xd3g Mac Maghnusa',
      'url': 'https://en.wikipedia.org/wiki/Cathal_%C3%93g_Mac_Maghnusa'},
     {'birth_year': u'1447',
      'gender': 'Male',
      'name': 'Philippe de Commines',
      'url': 'https://en.wikipedia.org/wiki/Philippe_de_Commines'},
     {'birth_year': u'1512',
      'gender': 'Male',
      'name': 'Robert Fabyan',
      'url': 'https://en.wikipedia.org/wiki/Robert_Fabyan'},
     {'birth_year': u'1450',
      'gender': 'Male',
      'name': 'Albert Krantz',
      'url': 'https://en.wikipedia.org/wiki/Albert_Krantz'},
     {'birth_year': u'1465',
      'gender': 'Male',
      'name': 'Hector Boece',
      'url': 'https://en.wikipedia.org/wiki/Hector_Boece'},
     {'birth_year': u'1470',
      'gender': 'Male',
      'name': 'Polydore Vergil',
      'url': 'https://en.wikipedia.org/wiki/Polydore_Vergil'},
     {'birth_year': u'1486',
      'gender': 'Male',
      'name': 'Sigismund von Herberstein',
      'url': 'https://en.wikipedia.org/wiki/Sigismund_von_Herberstein'},
     {'birth_year': u'1496',
      'gender': 'Male',
      'name': u'Jo\xe3o de Barros',
      'url': 'https://en.wikipedia.org/wiki/Jo%C3%A3o_de_Barros'},
     {'birth_year': u'1469',
      'gender': 'Male',
      'name': u'Niccol\xf2 Machiavelli',
      'url': 'https://en.wikipedia.org/wiki/Niccol%C3%B2_Machiavelli'},
     {'birth_year': u'1483',
      'gender': 'Male',
      'name': 'Francesco Guicciardini',
      'url': 'https://en.wikipedia.org/wiki/Francesco_Guicciardini'},
     {'birth_year': u'1530',
      'gender': 'Male',
      'name': 'Josias Simmler',
      'url': 'https://en.wikipedia.org/wiki/Josias_Simmler'},
     {'birth_year': u'1540',
      'gender': 'Male',
      'name': 'Paolo Paruta',
      'url': 'https://en.wikipedia.org/wiki/Paolo_Paruta'},
     {'birth_year': u'1529',
      'gender': 'Male',
      'name': 'Raphael Holinshed',
      'url': 'https://en.wikipedia.org/wiki/Raphael_Holinshed'},
     {'birth_year': u'1538',
      'gender': 'Male',
      'name': 'Caesar Baronius',
      'url': 'https://en.wikipedia.org/wiki/Caesar_Baronius'},
     {'birth_year': u'1540',
      'gender': 'Male',
      'name': "Abd al-Qadir Bada'uni",
      'url': 'https://en.wikipedia.org/wiki/Abd_al-Qadir_Bada%27uni'},
     {'birth_year': u'1549',
      'gender': '',
      'name': 'Abd al-Aziz al-Fishtali',
      'url': 'https://en.wikipedia.org/wiki/Abd_al-Aziz_al-Fishtali'},
     {'birth_year': u'1552',
      'gender': 'Male',
      'name': 'Ahmad Ibn al-Qadi',
      'url': 'https://en.wikipedia.org/wiki/Ahmad_Ibn_al-Qadi'},
     {'birth_year': u'1564',
      'gender': 'Male',
      'name': 'John Hayward',
      'url': 'https://en.wikipedia.org/wiki/John_Hayward_(historian)'},
     {'birth_year': u'1579',
      'gender': 'Male',
      'name': u'Pilip Ballach \xd3 Duibhgeann\xe1in',
      'url': 'https://en.wikipedia.org/wiki/Pilip_Ballach_%C3%93_Duibhgeann%C3%A1in'},
     {'birth_year': u'1581',
      'gender': 'Male',
      'name': 'James Ussher',
      'url': 'https://en.wikipedia.org/wiki/James_Ussher'},
     {'birth_year': None,
      'gender': '',
      'name': 'William Bradford',
      'url': 'https://en.wikipedia.org/wiki/William_Bradford_(Plymouth_governor)'},
     {'birth_year': u'1593',
      'gender': 'Male',
      'name': 'Bahrey',
      'url': 'https://en.wikipedia.org/wiki/Bahrey'},
     {'birth_year': u'1745',
      'gender': 'Male',
      'name': u'Fray \xcd\xf1igo Abbad y Lasierra',
      'url': 'https://en.wikipedia.org/wiki/Fray_%C3%8D%C3%B1igo_Abbad_y_Lasierra'},
     {'birth_year': '1797',
      'gender': 'Male',
      'name': 'Mohammed Akensus',
      'url': 'https://en.wikipedia.org/wiki/Mohammed_Akensus'},
     {'birth_year': u'1792',
      'gender': 'Male',
      'name': 'Archibald Alison',
      'url': 'https://en.wikipedia.org/wiki/Sir_Archibald_Alison,_1st_Baronet'},
     {'birth_year': u'1794',
      'gender': 'Male',
      'name': 'Abbasgulu Bakikhanov',
      'url': 'https://en.wikipedia.org/wiki/Abbasgulu_Bakikhanov'},
     {'birth_year': u'1782',
      'gender': 'Male',
      'name': 'Teimuraz Bagrationi',
      'url': 'https://en.wikipedia.org/wiki/Teimuraz_Bagrationi'},
     {'birth_year': u'1686',
      'gender': 'Male',
      'name': 'Archibald Bower',
      'url': 'https://en.wikipedia.org/wiki/Archibald_Bower'},
     {'birth_year': u'1645',
      'gender': 'Male',
      'name': u'\u0110or\u0111e Brankovi\u0107',
      'url': 'https://en.wikipedia.org/wiki/%C4%90or%C4%91e_Brankovi%C4%87_(count)'},
     {'birth_year': u'1610',
      'gender': 'Female',
      'name': 'Mary Bonaventure Browne',
      'url': 'https://en.wikipedia.org/wiki/Mary_Bonaventure_Browne'},
     {'birth_year': u'1666',
      'gender': 'Male',
      'name': 'Josiah Burchett',
      'url': 'https://en.wikipedia.org/wiki/Josiah_Burchett'},
     {'birth_year': '1949',
      'gender': 'Male',
      'name': 'Ron Chernow',
      'url': 'https://en.wikipedia.org/wiki/Ron_Chernow'},
     {'birth_year': u'1738',
      'gender': 'Male',
      'name': u"Chang Hs\xfceh-ch'eng",
      'url': 'https://en.wikipedia.org/wiki/Chang_Hs%C3%BCeh-ch%27eng'},
     {'birth_year': u'1795',
      'gender': 'Male',
      'name': 'Thomas Carlyle',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Carlyle'},
     {'birth_year': u'1891',
      'gender': 'Male',
      'name': 'Chaudhry Afzal Haq',
      'url': 'https://en.wikipedia.org/wiki/Chaudhry_Afzal_Haq'},
     {'birth_year': u'1917',
      'gender': 'Male',
      'name': 'Agha Shorish Kashmiri',
      'url': 'https://en.wikipedia.org/wiki/Agha_Shorish_Kashmiri'},
     {'birth_year': '1932',
      'gender': 'Male',
      'name': 'Janbaz Mirza',
      'url': 'https://en.wikipedia.org/wiki/Janbaz_Mirza'},
     {'birth_year': u'1793',
      'gender': 'Male',
      'name': 'Simonas Daukantas',
      'url': 'https://en.wikipedia.org/wiki/Simonas_Daukantas'},
     {'birth_year': u'1798',
      'gender': '',
      'name': 'Charles Dezobry',
      'url': 'https://en.wikipedia.org/wiki/Charles_Dezobry'},
     {'birth_year': u'1752',
      'gender': 'Male',
      'name': 'Mohammed al-Duayf',
      'url': 'https://en.wikipedia.org/wiki/Mohammed_al-Duayf'},
     {'birth_year': u'1785',
      'gender': 'Male',
      'name': 'John Colin Dunlop',
      'url': 'https://en.wikipedia.org/wiki/John_Colin_Dunlop'},
     {'birth_year': u'1670',
      'gender': 'Male',
      'name': 'Laurence Echard',
      'url': 'https://en.wikipedia.org/wiki/Laurence_Echard'},
     {'birth_year': u'1631',
      'gender': 'Male',
      'name': 'Abd al-Rahman al-Fasi',
      'url': 'https://en.wikipedia.org/wiki/Abd_al-Rahman_al-Fasi'},
     {'birth_year': u'1799',
      'gender': 'Male',
      'name': 'George Finlay',
      'url': 'https://en.wikipedia.org/wiki/George_Finlay'},
     {'birth_year': u'1549',
      'gender': '',
      'name': 'Abd al-Aziz al-Fishtali',
      'url': 'https://en.wikipedia.org/wiki/Abd_al-Aziz_al-Fishtali'},
     {'birth_year': u'1719',
      'gender': 'Male',
      'name': 'Francisco Jose Freire',
      'url': 'https://en.wikipedia.org/wiki/Francisco_Jose_Freire'},
     {'birth_year': u'1768',
      'gender': 'Male',
      'name': 'Francesco Maria Appendini',
      'url': 'https://en.wikipedia.org/wiki/Francesco_Maria_Appendini'},
     {'birth_year': u'1610',
      'gender': 'Male',
      'name': 'Charles du Fresne, sieur du Cange',
      'url': 'https://en.wikipedia.org/wiki/Charles_du_Fresne,_sieur_du_Cange'},
     {'birth_year': u'1698',
      'gender': 'Female',
      'name': u'Charlotta Fr\xf6lich',
      'url': 'https://en.wikipedia.org/wiki/Charlotta_Fr%C3%B6lich'},
     {'birth_year': u'1539',
      'gender': 'Male',
      'name': 'Garcilaso de la Vega',
      'url': 'https://en.wikipedia.org/wiki/Inca_Garcilaso_de_la_Vega'},
     {'birth_year': u'1783',
      'gender': 'Male',
      'name': 'Erik Gustaf Geijer',
      'url': 'https://en.wikipedia.org/wiki/Erik_Gustaf_Geijer'},
     {'birth_year': u'1737',
      'gender': 'Male',
      'name': 'Edward Gibbon',
      'url': 'https://en.wikipedia.org/wiki/Edward_Gibbon'},
     {'birth_year': u'1794',
      'gender': 'Male',
      'name': 'George Grote',
      'url': 'https://en.wikipedia.org/wiki/George_Grote'},
     {'birth_year': u'1668',
      'gender': 'Male',
      'name': 'Giambattista Vico',
      'url': 'https://en.wikipedia.org/wiki/Giambattista_Vico'},
     {'birth_year': u'1787',
      'gender': 'Male',
      'name': u'Fran\xe7ois Guizot',
      'url': 'https://en.wikipedia.org/wiki/Fran%C3%A7ois_Guizot'},
     {'birth_year': u'1732',
      'gender': 'Male',
      'name': 'Edward Hasted',
      'url': 'https://en.wikipedia.org/wiki/Edward_Hasted'},
     {'birth_year': u'1747',
      'gender': '',
      'name': 'Sulayman al-Hawwat',
      'url': 'https://en.wikipedia.org/wiki/Sulayman_al-Hawwat'},
     {'birth_year': u'1770',
      'gender': 'Male',
      'name': 'Georg Wilhelm Friedrich Hegel',
      'url': 'https://en.wikipedia.org/wiki/Georg_Wilhelm_Friedrich_Hegel'},
     {'birth_year': u'1739',
      'gender': 'Male',
      'name': 'Alexander Hewat (or Hewatt)',
      'url': 'https://en.wikipedia.org/wiki/Alexander_Hewat'},
     {'birth_year': u'1581',
      'gender': 'Male',
      'name': 'Pieter Corneliszoon Hooft',
      'url': 'https://en.wikipedia.org/wiki/Pieter_Corneliszoon_Hooft'},
     {'birth_year': u'1546',
      'gender': 'Male',
      'name': 'Arild Huitfeldt',
      'url': 'https://en.wikipedia.org/wiki/Arild_Huitfeldt'},
     {'birth_year': u'1711',
      'gender': 'Male',
      'name': 'David Hume',
      'url': 'https://en.wikipedia.org/wiki/David_Hume'},
     {'birth_year': u'1711',
      'gender': 'Male',
      'name': 'Thomas Hutchinson',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Hutchinson_(governor)'},
     {'birth_year': '1670',
      'gender': 'Male',
      'name': 'Mohammed al-Ifrani',
      'url': 'https://en.wikipedia.org/wiki/Mohammed_al-Ifrani'},
     {'birth_year': u'1766',
      'gender': 'Male',
      'name': 'Nikolai Mikhailovich Karamzin',
      'url': 'https://en.wikipedia.org/wiki/Nikolai_Mikhailovich_Karamzin'},
     {'birth_year': u'1569',
      'gender': 'Male',
      'name': 'Geoffrey Keating',
      'url': 'https://en.wikipedia.org/wiki/Geoffrey_Keating'},
     {'birth_year': u'1786',
      'gender': 'Male',
      'name': 'Joachim Lelewel',
      'url': 'https://en.wikipedia.org/wiki/Joachim_Lelewel'},
     {'birth_year': u'1771',
      'gender': 'Male',
      'name': 'John Lingard',
      'url': 'https://en.wikipedia.org/wiki/John_Lingard'},
     {'birth_year': u'1756',
      'gender': 'Male',
      'name': 'Anton Tomaz Linhart',
      'url': 'https://en.wikipedia.org/wiki/Anton_Tomaz_Linhart'},
     {'birth_year': u'1923',
      'gender': 'Male',
      'name': 'F.S.L. Lyons',
      'url': 'https://en.wikipedia.org/wiki/F.S.L._Lyons'},
     {'birth_year': u'1643',
      'gender': 'Male',
      'name': 'Dubhaltach MacFhirbhisigh',
      'url': 'https://en.wikipedia.org/wiki/Dubhaltach_MacFhirbhisigh'},
     {'birth_year': u'1798',
      'gender': 'Male',
      'name': 'Jules Michelet',
      'url': 'https://en.wikipedia.org/wiki/Jules_Michelet'},
     {'birth_year': u'1796',
      'gender': 'Male',
      'name': u'Fran\xe7ois Mignet',
      'url': 'https://en.wikipedia.org/wiki/Fran%C3%A7ois_Mignet'},
     {'birth_year': u'1783',
      'gender': 'Male',
      'name': 'Christian Molbech',
      'url': 'https://en.wikipedia.org/wiki/Christian_Molbech'},
     {'birth_year': u'1693',
      'gender': 'Male',
      'name': 'Johann Lorenz Von Mosheim',
      'url': 'https://en.wikipedia.org/wiki/Johann_Lorenz_Von_Mosheim'},
     {'birth_year': None,
      'gender': '',
      'name': u'Johannes von M\xfcller',
      'url': 'https://en.wikipedia.org/wiki/Johannes_von_M%C3%BCller'},
     {'birth_year': u'1672',
      'gender': 'Male',
      'name': 'Ludovico Antonio Muratori',
      'url': 'https://en.wikipedia.org/wiki/Ludovico_Antonio_Muratori'},
     {'birth_year': u'1637',
      'gender': 'Male',
      'name': u'Louis-S\xe9bastien Le Nain de Tillemont',
      'url': 'https://en.wikipedia.org/wiki/Louis-S%C3%A9bastien_Le_Nain_de_Tillemont'},
     {'birth_year': u'1776',
      'gender': 'Male',
      'name': 'Barthold Georg Niebuhr',
      'url': 'https://en.wikipedia.org/wiki/Barthold_Georg_Niebuhr'},
     {'birth_year': u'1614',
      'gender': 'Male',
      'name': u'Tadhg \xd3g \xd3 Cian\xe1in',
      'url': 'https://en.wikipedia.org/wiki/Tadhg_%C3%93g_%C3%93_Cian%C3%A1in'},
     {'birth_year': None,
      'gender': '',
      'name': u'M\xedche\xe1l \xd3 Cl\xe9irigh',
      'url': 'https://en.wikipedia.org/wiki/M%C3%ADche%C3%A1l_%C3%93_Cl%C3%A9irigh'},
     {'birth_year': u'1627',
      'gender': 'Male',
      'name': u'Peregrine \xd3 Duibhgeannain',
      'url': 'https://en.wikipedia.org/wiki/Peregrine_%C3%93_Duibhgeannain'},
     {'birth_year': u'1624',
      'gender': 'Male',
      'name': u'C\xfa Choigcr\xedche \xd3 Cl\xe9irigh',
      'url': 'https://en.wikipedia.org/wiki/C%C3%BA_Choigcr%C3%ADche_%C3%93_Cl%C3%A9irigh'},
     {'birth_year': u'1629',
      'gender': 'Male',
      'name': u'Ruaidhr\xed \xd3 Flaithbheartaigh',
      'url': 'https://en.wikipedia.org/wiki/Ruaidhr%C3%AD_%C3%93_Flaithbheartaigh'},
     {'birth_year': u'1726',
      'gender': 'Male',
      'name': 'Zaharije Orfelin',
      'url': 'https://en.wikipedia.org/wiki/Zaharije_Orfelin'},
     {'birth_year': None,
      'gender': '',
      'name': 'Olaus Magnus',
      'url': 'https://en.wikipedia.org/wiki/Olaus_Magnus'},
     {'birth_year': u'1798',
      'gender': 'Male',
      'name': u'Franti\u0161ek Palack\xfd',
      'url': 'https://en.wikipedia.org/wiki/Franti%C5%A1ek_Palack%C3%BD'},
     {'birth_year': u'1796',
      'gender': 'Male',
      'name': 'William H. Prescott',
      'url': 'https://en.wikipedia.org/wiki/William_H._Prescott'},
     {'birth_year': u'1609',
      'gender': 'Male',
      'name': 'Placido Puccinelli',
      'url': 'https://en.wikipedia.org/wiki/Placido_Puccinelli'},
     {'birth_year': u'1712',
      'gender': 'Male',
      'name': 'Mohammed al-Qadiri',
      'url': 'https://en.wikipedia.org/wiki/Mohammed_al-Qadiri'},
     {'birth_year': u'1728',
      'gender': 'Male',
      'name': 'Qian Daxin',
      'url': 'https://en.wikipedia.org/wiki/Qian_Daxin'},
     {'birth_year': u'1582',
      'gender': 'Male',
      'name': 'Qian Qianyi',
      'url': 'https://en.wikipedia.org/wiki/Qian_Qianyi'},
     {'birth_year': u'1749',
      'gender': 'Male',
      'name': 'David Ramsay',
      'url': 'https://en.wikipedia.org/wiki/David_Ramsay_(congressman)'},
     {'birth_year': u'1795',
      'gender': 'Male',
      'name': 'Leopold von Ranke',
      'url': 'https://en.wikipedia.org/wiki/Leopold_von_Ranke'},
     {'birth_year': '1938',
      'gender': 'Male',
      'name': 'John F. Richards',
      'url': 'https://en.wikipedia.org/wiki/John_F._Richards'},
     {'birth_year': u'1733',
      'gender': 'Male',
      'name': 'Mikhail Shcherbatov',
      'url': 'https://en.wikipedia.org/wiki/Mikhail_Shcherbatov'},
     {'birth_year': u'1643',
      'gender': 'Male',
      'name': 'John Strype',
      'url': 'https://en.wikipedia.org/wiki/John_Strype'},
     {'birth_year': u'1686',
      'gender': 'Male',
      'name': 'Vasily Tatishchev',
      'url': 'https://en.wikipedia.org/wiki/Vasily_Tatishchev'},
     {'birth_year': u'1797',
      'gender': 'Male',
      'name': 'Adolphe Thiers',
      'url': 'https://en.wikipedia.org/wiki/Adolphe_Thiers'},
     {'birth_year': u'1775',
      'gender': 'Male',
      'name': 'George Tucker',
      'url': 'https://en.wikipedia.org/wiki/George_Tucker_(politician)'},
     {'birth_year': u'1694',
      'gender': 'Male',
      'name': 'Voltaire',
      'url': 'https://en.wikipedia.org/wiki/Voltaire'},
     {'birth_year': u'1594',
      'gender': 'Male',
      'name': 'Sir James Ware',
      'url': 'https://en.wikipedia.org/wiki/Sir_James_Ware'},
     {'birth_year': u'1749',
      'gender': 'Male',
      'name': 'Yu Deuk-gong',
      'url': 'https://en.wikipedia.org/wiki/Yu_Deuk-gong'},
     {'birth_year': u'1734',
      'gender': 'Male',
      'name': 'Abu al-Qasim al-Zayyani',
      'url': 'https://en.wikipedia.org/wiki/Abu_al-Qasim_al-Zayyani'},
     {'birth_year': u'1672',
      'gender': 'Male',
      'name': 'Zhang Tingyu',
      'url': 'https://en.wikipedia.org/wiki/Zhang_Tingyu'},
     {'birth_year': u'1834',
      'gender': 'Male',
      'name': 'Lord Acton',
      'url': 'https://en.wikipedia.org/wiki/John_Dalberg-Acton,_1st_Baron_Acton'},
     {'birth_year': u'1838',
      'gender': 'Male',
      'name': 'Henry Adams',
      'url': 'https://en.wikipedia.org/wiki/Henry_Brooks_Adams'},
     {'birth_year': u'1816',
      'gender': 'Female',
      'name': 'Grace Aguilar',
      'url': 'https://en.wikipedia.org/wiki/Grace_Aguilar'},
     {'birth_year': u'1863',
      'gender': 'Male',
      'name': 'Charles McLean Andrews',
      'url': 'https://en.wikipedia.org/wiki/Charles_McLean_Andrews'},
     {'birth_year': u'1819',
      'gender': 'Male',
      'name': 'Alfred von Arneth',
      'url': 'https://en.wikipedia.org/wiki/Alfred_von_Arneth'},
     {'birth_year': u'1898',
      'gender': 'Male',
      'name': 'Mikhail Artamonov',
      'url': 'https://en.wikipedia.org/wiki/Mikhail_Artamonov'},
     {'birth_year': u'1860',
      'gender': 'Male',
      'name': 'William Ashley',
      'url': 'https://en.wikipedia.org/wiki/William_Ashley_(economic_historian)'},
     {'birth_year': u'1881',
      'gender': 'Male',
      'name': 'Octave Aubry',
      'url': 'https://en.wikipedia.org/wiki/Octave_Aubry'},
     {'birth_year': u'1849',
      'gender': 'Male',
      'name': u'Fran\xe7ois Victor Alphonse Aulard',
      'url': 'https://en.wikipedia.org/wiki/Fran%C3%A7ois_Victor_Alphonse_Aulard'},
     {'birth_year': u'1876',
      'gender': 'Male',
      'name': 'Zurab Avalishvili',
      'url': 'https://en.wikipedia.org/wiki/Zurab_Avalishvili'},
     {'birth_year': u'1879',
      'gender': 'Male',
      'name': 'Jacques Bainville',
      'url': 'https://en.wikipedia.org/wiki/Jacques_Bainville'},
     {'birth_year': u'1897',
      'gender': 'Female',
      'name': 'R. Mildred Barker',
      'url': 'https://en.wikipedia.org/wiki/R._Mildred_Barker'},
     {'birth_year': u'1889',
      'gender': 'Male',
      'name': 'Harry Elmer Barnes',
      'url': 'https://en.wikipedia.org/wiki/Harry_Elmer_Barnes'},
     {'birth_year': u'1879',
      'gender': 'Male',
      'name': 'Charles Bean',
      'url': 'https://en.wikipedia.org/wiki/Charles_Bean'},
     {'birth_year': u'1874',
      'gender': 'Male',
      'name': 'Charles A. Beard',
      'url': 'https://en.wikipedia.org/wiki/Charles_A._Beard'},
     {'birth_year': u'1876',
      'gender': 'Female',
      'name': 'Mary Ritter Beard',
      'url': 'https://en.wikipedia.org/wiki/Mary_Ritter_Beard'},
     {'birth_year': u'1800',
      'gender': 'Male',
      'name': 'George Bancroft',
      'url': 'https://en.wikipedia.org/wiki/George_Bancroft'},
     {'birth_year': u'1869',
      'gender': 'Male',
      'name': 'Wilhelm Barthold',
      'url': 'https://en.wikipedia.org/wiki/Vasily_Bartold'},
     {'birth_year': u'1884',
      'gender': 'Male',
      'name': 'Winthrop Pickard Bell',
      'url': 'https://en.wikipedia.org/wiki/Winthrop_Pickard_Bell'},
     {'birth_year': u'1870',
      'gender': 'Male',
      'name': 'Hilaire Belloc',
      'url': 'https://en.wikipedia.org/wiki/Hilaire_Belloc'},
     {'birth_year': u'1886',
      'gender': 'Male',
      'name': 'Marc Bloch',
      'url': 'https://en.wikipedia.org/wiki/Marc_Bloch'},
     {'birth_year': u'1870',
      'gender': 'Male',
      'name': 'Herbert Eugene Bolton',
      'url': 'https://en.wikipedia.org/wiki/Herbert_Eugene_Bolton'},
     {'birth_year': u'1894',
      'gender': 'Male',
      'name': 'George Williams Brown',
      'url': 'https://en.wikipedia.org/wiki/George_Williams_Brown'},
     {'birth_year': u'1868',
      'gender': '',
      'name': 'Erich Brandenburg',
      'url': 'https://en.wikipedia.org/wiki/Erich_Brandenburg'},
     {'birth_year': None,
      'gender': '',
      'name': 'Otto Brunner',
      'url': 'https://en.wikipedia.org/wiki/Otto_Brunner'},
     {'birth_year': u'1898',
      'gender': 'Male',
      'name': 'Geoffrey Bruun',
      'url': 'https://en.wikipedia.org/wiki/Geoffrey_Bruun'},
     {'birth_year': u'1899',
      'gender': 'Male',
      'name': 'Arthur Bryant',
      'url': 'https://en.wikipedia.org/wiki/Arthur_Bryant'},
     {'birth_year': u'1821',
      'gender': 'Male',
      'name': 'Henry Thomas Buckle',
      'url': 'https://en.wikipedia.org/wiki/Henry_Thomas_Buckle'},
     {'birth_year': u'1818',
      'gender': 'Male',
      'name': 'Jacob Burckhardt',
      'url': 'https://en.wikipedia.org/wiki/Jacob_Burckhardt'},
     {'birth_year': u'1809',
      'gender': 'Male',
      'name': 'John Hill Burton',
      'url': 'https://en.wikipedia.org/wiki/John_Hill_Burton'},
     {'birth_year': u'1861',
      'gender': 'Male',
      'name': 'J.B. Bury',
      'url': 'https://en.wikipedia.org/wiki/J.B._Bury'},
     {'birth_year': u'1885',
      'gender': 'Female',
      'name': 'Helen Cam',
      'url': 'https://en.wikipedia.org/wiki/Helen_Cam'},
     {'birth_year': u'1875',
      'gender': 'Male',
      'name': 'Pierre Caron',
      'url': 'https://en.wikipedia.org/wiki/Pierre_Caron_(historian)'},
     {'birth_year': u'1892',
      'gender': 'Male',
      'name': 'E.H. Carr',
      'url': 'https://en.wikipedia.org/wiki/E.H._Carr'},
     {'birth_year': u'1828',
      'gender': 'Male',
      'name': u'Antonio C\xe1novas del Castillo',
      'url': 'https://en.wikipedia.org/wiki/Antonio_C%C3%A1novas_del_Castillo'},
     {'birth_year': u'1831',
      'gender': 'Male',
      'name': 'Henri Raymond Casgrain',
      'url': 'https://en.wikipedia.org/wiki/Henri_Raymond_Casgrain'},
     {'birth_year': u'1885',
      'gender': 'Male',
      'name': u'Am\xe9rico Castro',
      'url': 'https://en.wikipedia.org/wiki/Am%C3%A9rico_Castro'},
     {'birth_year': u'1899',
      'gender': 'Male',
      'name': 'Bruce Catton',
      'url': 'https://en.wikipedia.org/wiki/Bruce_Catton'},
     {'birth_year': u'1810',
      'gender': 'Male',
      'name': 'Cesar de Bazancourt',
      'url': 'https://en.wikipedia.org/wiki/Baron_de_C%C3%A9sar_Bazancourt'},
     {'birth_year': u'1897',
      'gender': 'Male',
      'name': 'Nirad C. Chaudhuri',
      'url': 'https://en.wikipedia.org/wiki/Nirad_C._Chaudhuri'},
     {'birth_year': u'1828',
      'gender': 'Male',
      'name': 'Boris Chicherin',
      'url': 'https://en.wikipedia.org/wiki/Boris_Chicherin'},
     {'birth_year': u'1858',
      'gender': 'Male',
      'name': 'Hiram M. Chittenden',
      'url': 'https://en.wikipedia.org/wiki/Hiram_M._Chittenden'},
     {'birth_year': u'1874',
      'gender': 'Male',
      'name': 'Winston Churchill',
      'url': 'https://en.wikipedia.org/wiki/Winston_Churchill'},
     {'birth_year': u'1876',
      'gender': 'Male',
      'name': 'Augustin Cochin',
      'url': 'https://en.wikipedia.org/wiki/Augustin_Cochin_(historian)'},
     {'birth_year': u'1889',
      'gender': 'Male',
      'name': 'R. G. Collingwood',
      'url': 'https://en.wikipedia.org/wiki/R._G._Collingwood'},
     {'birth_year': u'1854',
      'gender': 'Male',
      'name': 'Julian Corbett',
      'url': 'https://en.wikipedia.org/wiki/Julian_Corbett'},
     {'birth_year': u'1885',
      'gender': 'Male',
      'name': u'Vladimir \u0106orovi\u0107',
      'url': 'https://en.wikipedia.org/wiki/Vladimir_%C4%86orovi%C4%87'},
     {'birth_year': u'1885',
      'gender': 'Male',
      'name': 'Avery Craven',
      'url': 'https://en.wikipedia.org/wiki/Avery_Craven'},
     {'birth_year': u'1812',
      'gender': 'Male',
      'name': 'Edward Shepherd Creasy',
      'url': 'https://en.wikipedia.org/wiki/Edward_Shepherd_Creasy'},
     {'birth_year': u'1894',
      'gender': 'Female',
      'name': 'Margaret Campbell Speke Cruwys',
      'url': 'https://en.wikipedia.org/wiki/Margaret_Campbell_Speke_Cruwys'},
     {'birth_year': u'1834',
      'gender': 'Male',
      'name': 'Felix Dahn',
      'url': 'https://en.wikipedia.org/wiki/Felix_Dahn'},
     {'birth_year': u'1890',
      'gender': 'Female',
      'name': 'Angie Debo',
      'url': 'https://en.wikipedia.org/wiki/Angie_Debo'},
     {'birth_year': u'1826',
      'gender': 'Male',
      'name': u'L\xe9opold Delisle',
      'url': 'https://en.wikipedia.org/wiki/L%C3%A9opold_Victor_Delisle'},
     {'birth_year': u'1897',
      'gender': 'Male',
      'name': 'Bernard DeVoto',
      'url': 'https://en.wikipedia.org/wiki/Bernard_DeVoto'},
     {'birth_year': None,
      'gender': '',
      'name': 'William Dodd',
      'url': 'https://en.wikipedia.org/wiki/William_Dodd_(ambassador)'},
     {'birth_year': u'1898',
      'gender': 'Male',
      'name': 'David C. Douglas',
      'url': 'https://en.wikipedia.org/wiki/David_C._Douglas'},
     {'birth_year': u'1808',
      'gender': 'Male',
      'name': 'Johann Gustav Droysen',
      'url': 'https://en.wikipedia.org/wiki/Johann_Gustav_Droysen'},
     {'birth_year': u'1898',
      'gender': 'Female',
      'name': 'Ariel Durant',
      'url': 'https://en.wikipedia.org/wiki/Ariel_Durant'},
     {'birth_year': u'1885',
      'gender': 'Male',
      'name': 'Will Durant',
      'url': 'https://en.wikipedia.org/wiki/Will_Durant'},
     {'birth_year': u'1818',
      'gender': 'Female',
      'name': 'Mary Anne Everett Green',
      'url': 'https://en.wikipedia.org/wiki/Mary_Anne_Everett_Green'},
     {'birth_year': u'1851',
      'gender': 'Male',
      'name': 'Ephraim Emerton',
      'url': 'https://en.wikipedia.org/wiki/Ephraim_Emerton'},
     {'birth_year': u'1888',
      'gender': 'Male',
      'name': 'Cyril Falls',
      'url': 'https://en.wikipedia.org/wiki/Cyril_Falls'},
     {'birth_year': u'1884',
      'gender': 'Male',
      'name': 'Keith Feiling',
      'url': 'https://en.wikipedia.org/wiki/Keith_Feiling'},
     {'birth_year': u'1893',
      'gender': 'Male',
      'name': 'Herbert Feis',
      'url': 'https://en.wikipedia.org/wiki/Herbert_Feis'},
     {'birth_year': u'1878',
      'gender': 'Male',
      'name': 'Lucien Febvre',
      'url': 'https://en.wikipedia.org/wiki/Lucien_Febvre'},
     {'birth_year': u'1857',
      'gender': 'Male',
      'name': 'Charles Harding Firth',
      'url': 'https://en.wikipedia.org/wiki/Charles_Harding_Firth'},
     {'birth_year': u'1874',
      'gender': 'Male',
      'name': 'Walter Lynwood Fleming',
      'url': 'https://en.wikipedia.org/wiki/Walter_Lynwood_Fleming'},
     {'birth_year': u'1823',
      'gender': 'Male',
      'name': 'Edward Augustus Freeman',
      'url': 'https://en.wikipedia.org/wiki/Edward_Augustus_Freeman'},
     {'birth_year': u'1818',
      'gender': 'Male',
      'name': 'James Anthony Froude',
      'url': 'https://en.wikipedia.org/wiki/James_Anthony_Froude'},
     {'birth_year': u'1878',
      'gender': 'Male',
      'name': 'J.F.C. Fuller',
      'url': 'https://en.wikipedia.org/wiki/J.F.C._Fuller'},
     {'birth_year': u'1862',
      'gender': 'Male',
      'name': 'Frantz Funck-Brentano',
      'url': 'https://en.wikipedia.org/wiki/Frantz_Funck-Brentano'},
     {'birth_year': u'1878',
      'gender': 'Male',
      'name': 'John Sydenham Furnivall',
      'url': 'https://en.wikipedia.org/wiki/John_Sydenham_Furnivall'},
     {'birth_year': u'1830',
      'gender': 'Male',
      'name': 'Numa Denis Fustel de Coulanges',
      'url': 'https://en.wikipedia.org/wiki/Numa_Denis_Fustel_de_Coulanges'},
     {'birth_year': u'1895',
      'gender': 'Male',
      'name': u'Fran\xe7ois-Louis Ganshof',
      'url': 'https://en.wikipedia.org/wiki/Fran%C3%A7ois-Louis_Ganshof'},
     {'birth_year': u'1829',
      'gender': 'Male',
      'name': 'Samuel Rawson Gardiner',
      'url': 'https://en.wikipedia.org/wiki/Samuel_Rawson_Gardiner'},
     {'birth_year': u'1887',
      'gender': 'Male',
      'name': 'Pieter Geyl',
      'url': 'https://en.wikipedia.org/wiki/Pieter_Geyl'},
     {'birth_year': u'1880',
      'gender': 'Male',
      'name': 'Lawrence Henry Gipson',
      'url': 'https://en.wikipedia.org/wiki/Lawrence_Henry_Gipson'},
     {'birth_year': u'1848',
      'gender': 'Male',
      'name': 'Arthur Giry',
      'url': 'https://en.wikipedia.org/wiki/Arthur_Giry'},
     {'birth_year': u'1862',
      'gender': 'Male',
      'name': 'Gustave Glotz',
      'url': 'https://en.wikipedia.org/wiki/Gustave_Glotz'},
     {'birth_year': u'1873',
      'gender': 'Male',
      'name': 'George Peabody Gooch',
      'url': 'https://en.wikipedia.org/wiki/George_Peabody_Gooch'},
     {'birth_year': u'1813',
      'gender': 'Male',
      'name': 'Timofey Granovsky',
      'url': 'https://en.wikipedia.org/wiki/Timofey_Granovsky'},
     {'birth_year': u'1837',
      'gender': 'Male',
      'name': 'John Richard Green',
      'url': 'https://en.wikipedia.org/wiki/John_Richard_Green'},
     {'birth_year': u'1878',
      'gender': 'Male',
      'name': 'Lionel Groulx',
      'url': 'https://en.wikipedia.org/wiki/Lionel_Groulx'},
     {'birth_year': u'1885',
      'gender': 'Male',
      'name': u'Ren\xe9 Grousset',
      'url': 'https://en.wikipedia.org/wiki/Ren%C3%A9_Grousset'},
     {'birth_year': u'1880',
      'gender': 'Male',
      'name': 'Louis Halphen',
      'url': 'https://en.wikipedia.org/wiki/Louis_Halphen'},
     {'birth_year': u'1885',
      'gender': 'Male',
      'name': 'Clarence H. Haring',
      'url': 'https://en.wikipedia.org/wiki/Clarence_H._Haring'},
     {'birth_year': u'1870',
      'gender': 'Male',
      'name': 'Charles H. Haskins',
      'url': 'https://en.wikipedia.org/wiki/Charles_H._Haskins'},
     {'birth_year': u'1866',
      'gender': 'Male',
      'name': 'Henri Hauser',
      'url': 'https://en.wikipedia.org/wiki/Henri_Hauser'},
     {'birth_year': None,
      'gender': '',
      'name': 'Julien Havet',
      'url': 'https://en.wikipedia.org/wiki/Julien_Havet'},
     {'birth_year': u'1878',
      'gender': 'Male',
      'name': 'Paul Hazard',
      'url': 'https://en.wikipedia.org/wiki/Paul_Hazard'},
     {'birth_year': u'1879',
      'gender': 'Male',
      'name': 'Eli Heckscher',
      'url': 'https://en.wikipedia.org/wiki/Eli_Heckscher'},
     {'birth_year': u'1823',
      'gender': 'Male',
      'name': 'Auguste Himly',
      'url': 'https://en.wikipedia.org/wiki/Auguste_Himly'},
     {'birth_year': u'1809',
      'gender': 'Male',
      'name': u'Mih\xe1ly Horv\xe1th',
      'url': 'https://en.wikipedia.org/wiki/Mih%C3%A1ly_Horv%C3%A1th'},
     {'birth_year': u'1872',
      'gender': 'Male',
      'name': 'Johan Huizinga',
      'url': 'https://en.wikipedia.org/wiki/Johan_Huizinga'},
     {'birth_year': u'1878',
      'gender': 'Male',
      'name': 'Ibn Zaydan',
      'url': 'https://en.wikipedia.org/wiki/Ibn_Zaydan'},
     {'birth_year': u'1832',
      'gender': 'Male',
      'name': 'Dmitry Ilovaisky',
      'url': 'https://en.wikipedia.org/wiki/Dmitry_Ilovaisky'},
     {'birth_year': u'1894',
      'gender': 'Male',
      'name': 'Harold Innis',
      'url': 'https://en.wikipedia.org/wiki/Harold_Innis'},
     {'birth_year': u'1858',
      'gender': '',
      'name': 'Mohammed ibn Jaafar al-Kattani',
      'url': 'https://en.wikipedia.org/wiki/Mohammed_ibn_Jaafar_al-Kattani'},
     {'birth_year': u'1875',
      'gender': 'Male',
      'name': 'Muhammad Jaber',
      'url': 'https://en.wikipedia.org/wiki/Muhammad_Jaber'},
     {'birth_year': u'1780',
      'gender': 'Male',
      'name': 'William James (naval historian)',
      'url': 'https://en.wikipedia.org/wiki/William_James_(naval_historian)'},
     {'birth_year': u'1876',
      'gender': 'Male',
      'name': 'Ivane Javakhishvili',
      'url': 'https://en.wikipedia.org/wiki/Ivane_Javakhishvili'},
     {'birth_year': None,
      'gender': '',
      'name': 'Samuel Kamakau',
      'url': 'https://en.wikipedia.org/wiki/Samuel_Kamakau'},
     {'birth_year': u'1818',
      'gender': 'Male',
      'name': 'Konstantin Kavelin',
      'url': 'https://en.wikipedia.org/wiki/Konstantin_Kavelin'},
     {'birth_year': u'1881',
      'gender': 'Male',
      'name': 'Hans Kelsen',
      'url': 'https://en.wikipedia.org/wiki/Hans_Kelsen'},
     {'birth_year': u'1855',
      'gender': 'Male',
      'name': 'Philip Moore Callow Kermode',
      'url': 'https://en.wikipedia.org/wiki/P._M._C._Kermode'},
     {'birth_year': u'1809',
      'gender': 'Male',
      'name': 'Alexander William Kinglake',
      'url': 'https://en.wikipedia.org/wiki/Alexander_William_Kinglake'},
     {'birth_year': u'1819',
      'gender': 'Male',
      'name': 'William Kingsford',
      'url': 'https://en.wikipedia.org/wiki/William_Kingsford'},
     {'birth_year': u'1841',
      'gender': 'Male',
      'name': 'Vasily Klyuchevsky',
      'url': 'https://en.wikipedia.org/wiki/Vasily_Klyuchevsky'},
     {'birth_year': u'1896',
      'gender': 'Male',
      'name': 'David Knowles',
      'url': 'https://en.wikipedia.org/wiki/David_Knowles_(scholar)'},
     {'birth_year': u'1877',
      'gender': 'Male',
      'name': 'Dudley Wright Knox',
      'url': 'https://en.wikipedia.org/wiki/Dudley_Wright_Knox'},
     {'birth_year': u'1800',
      'gender': 'Male',
      'name': u'Ludwig von K\xf6chel',
      'url': 'https://en.wikipedia.org/wiki/Ludwig_von_K%C3%B6chel'},
     {'birth_year': u'1817',
      'gender': 'Male',
      'name': u'Mihail Kog\u0103lniceanu',
      'url': 'https://en.wikipedia.org/wiki/Mihail_Kog%C4%83lniceanu'},
     {'birth_year': u'1891',
      'gender': 'Male',
      'name': 'Hans Kohn',
      'url': 'https://en.wikipedia.org/wiki/Hans_Kohn'},
     {'birth_year': u'1844',
      'gender': 'Male',
      'name': 'Nikodim Kondakov',
      'url': 'https://en.wikipedia.org/wiki/Nikodim_Kondakov'},
     {'birth_year': u'1817',
      'gender': 'Male',
      'name': 'Nikolay Kostomarov',
      'url': 'https://en.wikipedia.org/wiki/Nikolay_Kostomarov'},
     {'birth_year': u'1847',
      'gender': 'Male',
      'name': 'Godefroid Kurth',
      'url': 'https://en.wikipedia.org/wiki/Godefroid_Kurth'},
     {'birth_year': u'1897',
      'gender': 'Male',
      'name': 'Leonard Woods Labaree',
      'url': 'https://en.wikipedia.org/wiki/Leonard_Woods_Labaree'},
     {'birth_year': u'1896',
      'gender': 'Male',
      'name': 'William L. Langer',
      'url': 'https://en.wikipedia.org/wiki/William_L._Langer'},
     {'birth_year': u'1830',
      'gender': 'Male',
      'name': 'John Knox Laughton',
      'url': 'https://en.wikipedia.org/wiki/John_Knox_Laughton'},
     {'birth_year': u'1842',
      'gender': 'Male',
      'name': 'Ernest Lavisse',
      'url': 'https://en.wikipedia.org/wiki/Ernest_Lavisse'},
     {'birth_year': u'1874',
      'gender': 'Male',
      'name': 'Georges Lefebvre',
      'url': 'https://en.wikipedia.org/wiki/Georges_Lefebvre'},
     {'birth_year': u'1873',
      'gender': 'Male',
      'name': 'Liang Qichao',
      'url': 'https://en.wikipedia.org/wiki/Liang_Qichao'},
     {'birth_year': u'1895',
      'gender': 'Male',
      'name': 'B.H. Liddell Hart',
      'url': 'https://en.wikipedia.org/wiki/B.H._Liddell_Hart'},
     {'birth_year': u'1861',
      'gender': 'Male',
      'name': 'John Edward Lloyd',
      'url': 'https://en.wikipedia.org/wiki/John_Edward_Lloyd'},
     {'birth_year': u'1866',
      'gender': '',
      'name': 'Ferdinand Lot',
      'url': 'https://en.wikipedia.org/wiki/Ferdinand_Lot'},
     {'birth_year': u'1889',
      'gender': 'Male',
      'name': 'Arthur R.M. Lower',
      'url': 'https://en.wikipedia.org/wiki/Arthur_R.M._Lower'},
     {'birth_year': u'1800',
      'gender': 'Male',
      'name': 'Thomas Macaulay',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Macaulay'},
     {'birth_year': u'1895',
      'gender': 'Male',
      'name': 'William Archibald Mackintosh',
      'url': 'https://en.wikipedia.org/wiki/William_Archibald_Mackintosh'},
     {'birth_year': u'1887',
      'gender': 'Male',
      'name': 'J. D. Mackie',
      'url': 'https://en.wikipedia.org/wiki/J._D._Mackie'},
     {'birth_year': u'1850',
      'gender': 'Male',
      'name': 'Frederic William Maitland',
      'url': 'https://en.wikipedia.org/wiki/Frederic_William_Maitland'},
     {'birth_year': u'1840',
      'gender': 'Male',
      'name': 'Alfred Thayer Mahan',
      'url': 'https://en.wikipedia.org/wiki/Alfred_Thayer_Mahan'},
     {'birth_year': u'1888',
      'gender': 'Male',
      'name': 'Ramesh Chandra Majumdar',
      'url': 'https://en.wikipedia.org/wiki/Ramesh_Chandra_Majumdar'},
     {'birth_year': u'1859',
      'gender': 'Male',
      'name': 'J.A.R. Marriott',
      'url': 'https://en.wikipedia.org/wiki/John_Marriott_(British_politician)'},
     {'birth_year': u'1874',
      'gender': 'Male',
      'name': 'Albert Mathiez',
      'url': 'https://en.wikipedia.org/wiki/Albert_Mathiez'},
     {'birth_year': u'1818',
      'gender': 'Male',
      'name': 'Karl Marx',
      'url': 'https://en.wikipedia.org/wiki/Karl_Marx'},
     {'birth_year': u'1862',
      'gender': 'Male',
      'name': 'Friedrich Meinecke',
      'url': 'https://en.wikipedia.org/wiki/Friedrich_Meinecke'},
     {'birth_year': u'1874',
      'gender': 'Male',
      'name': 'Krste Misirkov',
      'url': 'https://en.wikipedia.org/wiki/Krste_Misirkov'},
     {'birth_year': u'1851',
      'gender': 'Male',
      'name': 'Auguste Molinier',
      'url': 'https://en.wikipedia.org/wiki/Auguste_Molinier'},
     {'birth_year': u'1817',
      'gender': 'Male',
      'name': 'Theodor Mommsen',
      'url': 'https://en.wikipedia.org/wiki/Theodor_Mommsen'},
     {'birth_year': u'1909',
      'gender': 'Male',
      'name': 'Indro Montanelli',
      'url': 'https://en.wikipedia.org/wiki/Indro_Montanelli'},
     {'birth_year': u'1850',
      'gender': 'Male',
      'name': 'Alfred Morel-Fatio',
      'url': 'https://en.wikipedia.org/wiki/Alfred_Morel-Fatio'},
     {'birth_year': u'1887',
      'gender': 'Male',
      'name': 'Samuel Eliot Morison',
      'url': 'https://en.wikipedia.org/wiki/Samuel_Eliot_Morison'},
     {'birth_year': u'1895',
      'gender': 'Male',
      'name': 'Lewis Mumford',
      'url': 'https://en.wikipedia.org/wiki/Lewis_Mumford'},
     {'birth_year': u'1888',
      'gender': 'Male',
      'name': 'Lewis Bernstein Namier',
      'url': 'https://en.wikipedia.org/wiki/Lewis_Bernstein_Namier'},
     {'birth_year': u'1834',
      'gender': 'Male',
      'name': 'Ahmad ibn Khalid al-Nasiri',
      'url': 'https://en.wikipedia.org/wiki/Ahmad_ibn_Khalid_al-Nasiri'},
     {'birth_year': u'1890',
      'gender': 'Male',
      'name': 'J. E. Neale',
      'url': 'https://en.wikipedia.org/wiki/J._E._Neale'},
     {'birth_year': u'1890',
      'gender': 'Male',
      'name': 'Allan Nevins',
      'url': 'https://en.wikipedia.org/wiki/Allan_Nevins'},
     {'birth_year': None,
      'gender': '',
      'name': 'A. P. Newton',
      'url': 'https://en.wikipedia.org/wiki/A._P._Newton'},
     {'birth_year': u'1842',
      'gender': 'Male',
      'name': u'Stojan Novakovi\u0107',
      'url': 'https://en.wikipedia.org/wiki/Stojan_Novakovi%C4%87'},
     {'birth_year': u'1860',
      'gender': 'Male',
      'name': 'Charles Oman',
      'url': 'https://en.wikipedia.org/wiki/Charles_Oman'},
     {'birth_year': u'1855',
      'gender': 'Male',
      'name': 'Herbert L. Osgood',
      'url': 'https://en.wikipedia.org/wiki/Herbert_L._Osgood'},
     {'birth_year': u'1840',
      'gender': 'Male',
      'name': 'Cesare Paoli',
      'url': 'https://en.wikipedia.org/wiki/Cesare_Paoli'},
     {'birth_year': u'1839',
      'gender': 'Male',
      'name': 'Gaston Paris',
      'url': 'https://en.wikipedia.org/wiki/Gaston_Paris'},
     {'birth_year': u'1853',
      'gender': 'Male',
      'name': 'Herbert Paul',
      'url': 'https://en.wikipedia.org/wiki/Herbert_Paul'},
     {'birth_year': u'1846',
      'gender': 'Male',
      'name': 'Henry Francis Pelham',
      'url': 'https://en.wikipedia.org/wiki/Henry_Francis_Pelham'},
     {'birth_year': u'1843',
      'gender': 'Male',
      'name': 'Samuel W. Pennypacker',
      'url': 'https://en.wikipedia.org/wiki/Samuel_W._Pennypacker'},
     {'birth_year': u'1889',
      'gender': 'Male',
      'name': 'Dexter Perkins',
      'url': 'https://en.wikipedia.org/wiki/Dexter_Perkins'},
     {'birth_year': None,
      'gender': '',
      'name': 'Henri Pirenne',
      'url': 'https://en.wikipedia.org/wiki/Henri_Pirenne'},
     {'birth_year': u'1860',
      'gender': 'Male',
      'name': 'Sergey Platonov',
      'url': 'https://en.wikipedia.org/wiki/Sergey_Platonov'},
     {'birth_year': u'1898',
      'gender': 'Female',
      'name': 'Ivy Pinchbeck',
      'url': 'https://en.wikipedia.org/wiki/Ivy_Pinchbeck'},
     {'birth_year': u'1889',
      'gender': 'Female',
      'name': 'Eileen Power',
      'url': 'https://en.wikipedia.org/wiki/Eileen_Power'},
     {'birth_year': u'1879',
      'gender': 'Male',
      'name': 'F. M. Powicke',
      'url': 'https://en.wikipedia.org/wiki/F._M._Powicke'},
     {'birth_year': u'1896',
      'gender': 'Female',
      'name': 'H. F. M. Prescott',
      'url': 'https://en.wikipedia.org/wiki/H._F._M._Prescott'},
     {'birth_year': u'1890',
      'gender': 'Male',
      'name': 'Datto Vaman Potdar',
      'url': 'https://en.wikipedia.org/wiki/Datto_Vaman_Potdar'},
     {'birth_year': u'1814',
      'gender': 'Male',
      'name': 'Jules Quicherat',
      'url': 'https://en.wikipedia.org/wiki/Jules_Etienne_Joseph_Quicherat'},
     {'birth_year': u'1857',
      'gender': 'Male',
      'name': 'William Pember Reeves',
      'url': 'https://en.wikipedia.org/wiki/William_Pember_Reeves'},
     {'birth_year': u'1893',
      'gender': 'Male',
      'name': 'Pierre Renouvin',
      'url': 'https://en.wikipedia.org/wiki/Pierre_Renouvin'},
     {'birth_year': u'1822',
      'gender': 'Male',
      'name': 'James Riker',
      'url': 'https://en.wikipedia.org/wiki/James_Riker'},
     {'birth_year': u'1857',
      'gender': 'Male',
      'name': 'B. H. Roberts',
      'url': 'https://en.wikipedia.org/wiki/B._H._Roberts'},
     {'birth_year': None,
      'gender': '',
      'name': 'James Harvey Robinson',
      'url': 'https://en.wikipedia.org/wiki/James_Harvey_Robinson'},
     {'birth_year': u'1858',
      'gender': 'Male',
      'name': 'Theodore Roosevelt',
      'url': 'https://en.wikipedia.org/wiki/Theodore_Roosevelt'},
     {'birth_year': u'1851',
      'gender': 'Male',
      'name': 'Simon Rutar',
      'url': 'https://en.wikipedia.org/wiki/Simon_Rutar'},
     {'birth_year': u'1832',
      'gender': 'Male',
      'name': 'Ilarion Ruvarac',
      'url': 'https://en.wikipedia.org/wiki/Ilarion_Ruvarac'},
     {'birth_year': u'1899',
      'gender': 'Male',
      'name': 'Abram L. Sachar',
      'url': 'https://en.wikipedia.org/wiki/Abram_L._Sachar'},
     {'birth_year': u'1884',
      'gender': 'Male',
      'name': 'George Sarton',
      'url': 'https://en.wikipedia.org/wiki/George_Sarton'},
     {'birth_year': u'1844',
      'gender': 'Male',
      'name': 'Gustave Schlumberger',
      'url': 'https://en.wikipedia.org/wiki/Gustave_Schlumberger'},
     {'birth_year': u'1834',
      'gender': 'Male',
      'name': 'John Robert Seeley',
      'url': 'https://en.wikipedia.org/wiki/John_Robert_Seeley'},
     {'birth_year': u'1820',
      'gender': 'Male',
      'name': 'Sergey Solovyov',
      'url': 'https://en.wikipedia.org/wiki/Sergey_Solovyov'},
     {'birth_year': u'1865',
      'gender': 'Male',
      'name': 'Govind Sakharam Sardesai',
      'url': 'https://en.wikipedia.org/wiki/Govind_Sakharam_Sardesai'},
     {'birth_year': u'1859',
      'gender': 'Male',
      'name': 'Adam Shortt',
      'url': 'https://en.wikipedia.org/wiki/Adam_Shortt'},
     {'birth_year': u'1823',
      'gender': 'Male',
      'name': 'Goldwin Smith',
      'url': 'https://en.wikipedia.org/wiki/Goldwin_Smith'},
     {'birth_year': u'1880',
      'gender': 'Male',
      'name': 'Oswald Spengler',
      'url': 'https://en.wikipedia.org/wiki/Oswald_Spengler'},
     {'birth_year': u'1880',
      'gender': 'Male',
      'name': 'Shin Chaeho',
      'url': 'https://en.wikipedia.org/wiki/Shin_Chaeho'},
     {'birth_year': u'1880',
      'gender': 'Male',
      'name': 'Frank Stenton',
      'url': 'https://en.wikipedia.org/wiki/Frank_Stenton'},
     {'birth_year': u'1894',
      'gender': 'Female',
      'name': 'Doris Mary Stenton',
      'url': 'https://en.wikipedia.org/wiki/Doris_Mary_Stenton'},
     {'birth_year': u'1825',
      'gender': 'Male',
      'name': 'William Stubbs',
      'url': 'https://en.wikipedia.org/wiki/William_Stubbs'},
     {'birth_year': u'1828',
      'gender': 'Male',
      'name': 'Hippolyte Taine',
      'url': 'https://en.wikipedia.org/wiki/Hippolyte_Taine'},
     {'birth_year': u'1853',
      'gender': 'Male',
      'name': 'Frank Bigelow Tarbell',
      'url': 'https://en.wikipedia.org/wiki/Frank_Bigelow_Tarbell'},
     {'birth_year': u'1874',
      'gender': 'Male',
      'name': 'Yevgeny Tarle',
      'url': 'https://en.wikipedia.org/wiki/Yevgeny_Tarle'},
     {'birth_year': u'1880',
      'gender': 'Male',
      'name': 'A. Wyatt Tilby',
      'url': 'https://en.wikipedia.org/wiki/A._Wyatt_Tilby'},
     {'birth_year': u'1805',
      'gender': 'Male',
      'name': 'Alexis de Tocqueville',
      'url': 'https://en.wikipedia.org/wiki/Alexis_de_Tocqueville'},
     {'birth_year': u'1818',
      'gender': 'Male',
      'name': 'Zacharias Topelius',
      'url': 'https://en.wikipedia.org/wiki/Zacharias_Topelius'},
     {'birth_year': u'1855',
      'gender': 'Male',
      'name': 'Thomas Frederick Tout',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Frederick_Tout'},
     {'birth_year': u'1889',
      'gender': 'Male',
      'name': 'Arnold J. Toynbee',
      'url': 'https://en.wikipedia.org/wiki/Arnold_J._Toynbee'},
     {'birth_year': u'1834',
      'gender': 'Male',
      'name': 'Heinrich Gotthard von Treitschke',
      'url': 'https://en.wikipedia.org/wiki/Heinrich_Gotthard_von_Treitschke'},
     {'birth_year': u'1876',
      'gender': 'Male',
      'name': 'George Macaulay Trevelyan',
      'url': 'https://en.wikipedia.org/wiki/George_Macaulay_Trevelyan'},
     {'birth_year': u'1878',
      'gender': 'Male',
      'name': 'Mikheil Tsereteli',
      'url': 'https://en.wikipedia.org/wiki/Mikheil_Tsereteli'},
     {'birth_year': u'1889',
      'gender': 'Male',
      'name': 'Frank Underhill',
      'url': 'https://en.wikipedia.org/wiki/Frank_Underhill'},
     {'birth_year': u'1854',
      'gender': 'Male',
      'name': 'Paul Vinogradoff',
      'url': 'https://en.wikipedia.org/wiki/Paul_Vinogradoff'},
     {'birth_year': u'1839',
      'gender': 'Male',
      'name': 'Spencer Walpole',
      'url': 'https://en.wikipedia.org/wiki/Spencer_Walpole'},
     {'birth_year': u'1886',
      'gender': 'Male',
      'name': 'Curt Weibull',
      'url': 'https://en.wikipedia.org/wiki/Curt_Weibull'},
     {'birth_year': u'1873',
      'gender': 'Male',
      'name': 'Lauritz Weibull',
      'url': 'https://en.wikipedia.org/wiki/Lauritz_Weibull'},
     {'birth_year': u'1886',
      'gender': 'Male',
      'name': 'Charles Webster',
      'url': 'https://en.wikipedia.org/wiki/Charles_Webster_(historian)'},
     {'birth_year': u'1878',
      'gender': 'Female',
      'name': 'Mary Wilhelmine Williams',
      'url': 'https://en.wikipedia.org/wiki/Mary_Wilhelmine_Williams'},
     {'birth_year': u'1853',
      'gender': 'Male',
      'name': 'Spenser Wilkinson',
      'url': 'https://en.wikipedia.org/wiki/Spenser_Wilkinson'},
     {'birth_year': u'1886',
      'gender': 'Male',
      'name': 'James A. Williamson',
      'url': 'https://en.wikipedia.org/wiki/James_Williamson_(historian)'},
     {'birth_year': u'1831',
      'gender': 'Male',
      'name': 'Justin Winsor',
      'url': 'https://en.wikipedia.org/wiki/Justin_Winsor'},
     {'birth_year': u'1890',
      'gender': 'Male',
      'name': 'Llewellyn Woodward',
      'url': 'https://en.wikipedia.org/wiki/Llewellyn_Woodward'},
     {'birth_year': u'1860',
      'gender': 'Male',
      'name': 'George MacKinnon Wrong',
      'url': 'https://en.wikipedia.org/wiki/George_MacKinnon_Wrong'},
     {'birth_year': u'1896',
      'gender': 'Male',
      'name': 'Yi Byeongdo',
      'url': 'https://en.wikipedia.org/wiki/Yi_Byeongdo'},
     {'birth_year': u'1859',
      'gender': '',
      'name': 'Faddei Zielinski',
      'url': 'https://en.wikipedia.org/wiki/Faddei_Zielinski'},
     {'birth_year': u'1939',
      'gender': '',
      'name': 'Raouf Abbas',
      'url': 'https://en.wikipedia.org/wiki/Raouf_Abbas'},
     {'birth_year': '1940',
      'gender': 'Male',
      'name': 'Irving Abella',
      'url': 'https://en.wikipedia.org/wiki/Irving_Abella'},
     {'birth_year': u'2004',
      'gender': 'Male',
      'name': 'Aberjhani',
      'url': 'https://en.wikipedia.org/wiki/Aberjhani'},
     {'birth_year': '1949',
      'gender': 'Male',
      'name': 'David Abulafia',
      'url': 'https://en.wikipedia.org/wiki/David_Abulafia'},
     {'birth_year': u'1971',
      'gender': 'Male',
      'name': 'Ezequiel Adamovsky',
      'url': 'https://en.wikipedia.org/wiki/Ezequiel_Adamovsky'},
     {'birth_year': u'1939',
      'gender': 'Male',
      'name': 'Donald Adamson',
      'url': 'https://en.wikipedia.org/wiki/Donald_Adamson'},
     {'birth_year': u'1912',
      'gender': 'Male',
      'name': 'Teodoro Agoncillo',
      'url': 'https://en.wikipedia.org/wiki/Teodoro_Agoncillo'},
     {'birth_year': u'1896',
      'gender': 'Male',
      'name': 'Robert G. Albion',
      'url': 'https://en.wikipedia.org/wiki/Robert_G._Albion'},
     {'birth_year': '1933',
      'gender': 'Male',
      'name': 'Dean C. Allard',
      'url': 'https://en.wikipedia.org/wiki/Dean_C._Allard'},
     {'birth_year': '1947',
      'gender': 'Male',
      'name': 'Robert C. Allen',
      'url': 'https://en.wikipedia.org/wiki/Robert_C._Allen'},
     {'birth_year': u'1936',
      'gender': 'Male',
      'name': 'Gar Alperovitz',
      'url': 'https://en.wikipedia.org/wiki/Gar_Alperovitz'},
     {'birth_year': u'1950',
      'gender': 'Female',
      'name': 'Ida Altman',
      'url': 'https://en.wikipedia.org/wiki/Ida_Altman'},
     {'birth_year': u'1947',
      'gender': 'Male',
      'name': 'Abbas Amanat',
      'url': 'https://en.wikipedia.org/wiki/Abbas_Amanat'},
     {'birth_year': u'1957',
      'gender': 'Female',
      'name': 'Mor Altshuler',
      'url': 'https://en.wikipedia.org/wiki/Mor_Altshuler'},
     {'birth_year': u'1920',
      'gender': 'Male',
      'name': 'Henri Amouroux',
      'url': 'https://en.wikipedia.org/wiki/Henri_Amouroux'},
     {'birth_year': u'1936',
      'gender': 'Male',
      'name': 'Stephen Ambrose',
      'url': 'https://en.wikipedia.org/wiki/Stephen_Ambrose'},
     {'birth_year': None,
      'gender': '',
      'name': 'Perry Anderson',
      'url': 'https://en.wikipedia.org/wiki/Perry_Anderson'},
     {'birth_year': u'1929',
      'gender': 'Female',
      'name': 'Joyce Appleby',
      'url': 'https://en.wikipedia.org/wiki/Joyce_Appleby'},
     {'birth_year': u'1915',
      'gender': 'Male',
      'name': 'Herbert Aptheker',
      'url': 'https://en.wikipedia.org/wiki/Herbert_Aptheker'},
     {'birth_year': '1955',
      'gender': '',
      'name': 'Leonie Archer',
      'url': 'https://en.wikipedia.org/wiki/Leonie_Archer'},
     {'birth_year': u'1914',
      'gender': 'Male',
      'name': u'Philippe Ari\xe8s',
      'url': 'https://en.wikipedia.org/wiki/Philippe_Ari%C3%A8s'},
     {'birth_year': None,
      'gender': '',
      'name': 'Karen Armstrong',
      'url': 'https://en.wikipedia.org/wiki/Karen_Armstrong'},
     {'birth_year': u'1917',
      'gender': 'Male',
      'name': 'Leonard J. Arrington',
      'url': 'https://en.wikipedia.org/wiki/Leonard_J._Arrington'},
     {'birth_year': '1999',
      'gender': 'Male',
      'name': 'Thomas Asbridge',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Asbridge'},
     {'birth_year': u'1907',
      'gender': 'Male',
      'name': 'Maurice Ashley',
      'url': 'https://en.wikipedia.org/wiki/Maurice_Ashley_(historian)'},
     {'birth_year': u'1931',
      'gender': 'Male',
      'name': 'Paul Avrich',
      'url': 'https://en.wikipedia.org/wiki/Paul_Avrich'},
     {'birth_year': u'1942',
      'gender': 'Male',
      'name': 'Ali Azaykou',
      'url': 'https://en.wikipedia.org/wiki/Ali_Azaykou'},
     {'birth_year': '1966',
      'gender': 'Male',
      'name': 'Eiichiro Azuma',
      'url': 'https://en.wikipedia.org/wiki/Eiichiro_Azuma'},
     {'birth_year': u'1927',
      'gender': 'Male',
      'name': 'Nigel Bagnall',
      'url': 'https://en.wikipedia.org/wiki/Nigel_Bagnall'},
     {'birth_year': u'1922',
      'gender': 'Male',
      'name': 'Bernard Bailyn',
      'url': 'https://en.wikipedia.org/wiki/Bernard_Bailyn'},
     {'birth_year': '1948',
      'gender': 'Male',
      'name': 'David E. Barclay',
      'url': 'https://en.wikipedia.org/wiki/David_E._Barclay'},
     {'birth_year': u'1958',
      'gender': 'Female',
      'name': 'Juliet Barker',
      'url': 'https://en.wikipedia.org/wiki/Juliet_Barker'},
     {'birth_year': u'1911',
      'gender': 'Male',
      'name': 'Frank Barlow',
      'url': 'https://en.wikipedia.org/wiki/Frank_Barlow_(historian)'},
     {'birth_year': None,
      'gender': 'Female',
      'name': 'Linda Diane Barnes',
      'url': 'https://en.wikipedia.org/wiki/Linda_Diane_Barnes'},
     {'birth_year': u'1924',
      'gender': 'Male',
      'name': 'G.W.S. Barrow',
      'url': 'https://en.wikipedia.org/wiki/G.W.S._Barrow'},
     {'birth_year': u'1929',
      'gender': 'Male',
      'name': 'H. Arnold Barton',
      'url': 'https://en.wikipedia.org/wiki/H._Arnold_Barton'},
     {'birth_year': u'1955',
      'gender': 'Male',
      'name': 'Paul R. Bartrop',
      'url': 'https://en.wikipedia.org/wiki/Paul_R._Bartrop'},
     {'birth_year': u'1907',
      'gender': 'Male',
      'name': 'Jacques Barzun',
      'url': 'https://en.wikipedia.org/wiki/Jacques_Barzun'},
     {'birth_year': u'1903',
      'gender': 'Male',
      'name': 'Jorge Basadre',
      'url': 'https://en.wikipedia.org/wiki/Jorge_Basadre'},
     {'birth_year': u'1926',
      'gender': 'Male',
      'name': 'Hanna Batatu',
      'url': 'https://en.wikipedia.org/wiki/Hanna_Batatu'},
     {'birth_year': u'1926',
      'gender': 'Male',
      'name': 'K. Jack Bauer',
      'url': 'https://en.wikipedia.org/wiki/K._Jack_Bauer'},
     {'birth_year': u'1926',
      'gender': 'Male',
      'name': 'Yehuda Bauer',
      'url': 'https://en.wikipedia.org/wiki/Yehuda_Bauer'},
     {'birth_year': '1959',
      'gender': 'Male',
      'name': 'Stephen B. Baxter',
      'url': 'https://en.wikipedia.org/wiki/Stephen_B._Baxter'},
     {'birth_year': u'1949',
      'gender': 'Male',
      'name': 'David Bebbington',
      'url': 'https://en.wikipedia.org/wiki/David_Bebbington'},
     {'birth_year': u'1946',
      'gender': 'Male',
      'name': 'Antony Beevor',
      'url': 'https://en.wikipedia.org/wiki/Antony_Beevor'},
     {'birth_year': u'1956',
      'gender': 'Male',
      'name': 'James Belich',
      'url': 'https://en.wikipedia.org/wiki/James_Belich_(historian)'},
     {'birth_year': u'1919',
      'gender': 'Male',
      'name': 'Abdelmajid Benjelloun',
      'url': 'https://en.wikipedia.org/wiki/Abdelmajid_Benjelloun_(historian)'},
     {'birth_year': u'1950',
      'gender': 'Male',
      'name': 'Laurence Bergreen',
      'url': 'https://en.wikipedia.org/wiki/Laurence_Bergreen'},
     {'birth_year': u'1909',
      'gender': 'Male',
      'name': 'Isaiah Berlin',
      'url': 'https://en.wikipedia.org/wiki/Isaiah_Berlin'},
     {'birth_year': u'1955',
      'gender': 'Male',
      'name': 'Michael Beschloss',
      'url': 'https://en.wikipedia.org/wiki/Michael_Beschloss'},
     {'birth_year': u'1938',
      'gender': 'Male',
      'name': 'Nicholas Bethell',
      'url': 'https://en.wikipedia.org/wiki/Nicholas_Bethell'},
     {'birth_year': u'1937',
      'gender': 'Male',
      'name': 'Anthony Birley',
      'url': 'https://en.wikipedia.org/wiki/Anthony_Birley'},
     {'birth_year': u'1949',
      'gender': 'Male',
      'name': 'David Blackbourn',
      'url': 'https://en.wikipedia.org/wiki/David_Blackbourn'},
     {'birth_year': u'1930',
      'gender': 'Male',
      'name': 'Geoffrey Blainey',
      'url': 'https://en.wikipedia.org/wiki/Geoffrey_Blainey'},
     {'birth_year': u'1942',
      'gender': 'Female',
      'name': 'Gisela Bock',
      'url': 'https://en.wikipedia.org/wiki/Gisela_Bock'},
     {'birth_year': u'1936',
      'gender': 'Male',
      'name': 'Brian Bond',
      'url': 'https://en.wikipedia.org/wiki/Brian_Bond'},
     {'birth_year': u'1914',
      'gender': 'Male',
      'name': 'Daniel J. Boorstin',
      'url': 'https://en.wikipedia.org/wiki/Daniel_J._Boorstin'},
     {'birth_year': u'1920',
      'gender': 'Male',
      'name': 'Georges Bordonove',
      'url': 'https://en.wikipedia.org/wiki/Georges_Bordonove'},
     {'birth_year': u'1947',
      'gender': 'Male',
      'name': 'John Boswell',
      'url': 'https://en.wikipedia.org/wiki/John_Boswell'},
     {'birth_year': '1944',
      'gender': 'Male',
      'name': 'Robert Bothwell',
      'url': 'https://en.wikipedia.org/wiki/Robert_Bothwell'},
     {'birth_year': u'1943',
      'gender': 'Male',
      'name': u'G\xe9rard Bouchard',
      'url': 'https://en.wikipedia.org/wiki/G%C3%A9rard_Bouchard'},
     {'birth_year': '1963',
      'gender': 'Female',
      'name': 'Joanna Bourke',
      'url': 'https://en.wikipedia.org/wiki/Joanna_Bourke'},
     {'birth_year': u'1935',
      'gender': 'Male',
      'name': 'Paul S. Boyer',
      'url': 'https://en.wikipedia.org/wiki/Paul_S._Boyer'},
     {'birth_year': u'1922',
      'gender': 'Male',
      'name': 'Karl Dietrich Bracher',
      'url': 'https://en.wikipedia.org/wiki/Karl_Dietrich_Bracher'},
     {'birth_year': '1937',
      'gender': '',
      'name': 'Jim Bradbury',
      'url': 'https://en.wikipedia.org/wiki/Jim_Bradbury'},
     {'birth_year': u'1945',
      'gender': 'Male',
      'name': 'James C. Bradford',
      'url': 'https://en.wikipedia.org/wiki/James_C._Bradford'},
     {'birth_year': u'1936',
      'gender': 'Male',
      'name': 'David Brading',
      'url': 'https://en.wikipedia.org/wiki/David_Brading'},
     {'birth_year': u'1914',
      'gender': 'Male',
      'name': 'William Brandon',
      'url': 'https://en.wikipedia.org/wiki/William_Brandon_(author)'},
     {'birth_year': u'1902',
      'gender': 'Male',
      'name': 'Fernand Braudel',
      'url': 'https://en.wikipedia.org/wiki/Fernand_Braudel'},
     {'birth_year': u'1958',
      'gender': 'Male',
      'name': 'Ahron Bregman',
      'url': 'https://en.wikipedia.org/wiki/Ahron_Bregman'},
     {'birth_year': u'1903',
      'gender': 'Male',
      'name': 'Carl Bridenbaugh',
      'url': 'https://en.wikipedia.org/wiki/Carl_Bridenbaugh'},
     {'birth_year': u'1951',
      'gender': 'Male',
      'name': 'Timothy Brook',
      'url': 'https://en.wikipedia.org/wiki/Timothy_Brook_(historian)'},
     {'birth_year': u'1926',
      'gender': 'Male',
      'name': 'Martin Broszat',
      'url': 'https://en.wikipedia.org/wiki/Martin_Broszat'},
     {'birth_year': None,
      'gender': '',
      'name': 'Peter Brown',
      'url': 'https://en.wikipedia.org/wiki/Peter_Brown_(historian)'},
     {'birth_year': u'1944',
      'gender': 'Male',
      'name': 'Christopher Browning',
      'url': 'https://en.wikipedia.org/wiki/Christopher_Browning'},
     {'birth_year': u'1914',
      'gender': 'Male',
      'name': 'Alan Bullock',
      'url': 'https://en.wikipedia.org/wiki/Alan_Bullock'},
     {'birth_year': u'1937',
      'gender': 'Male',
      'name': 'Peter Burke',
      'url': 'https://en.wikipedia.org/wiki/Peter_Burke_(historian)'},
     {'birth_year': u'1936',
      'gender': 'Male',
      'name': 'Briton C. Busch',
      'url': 'https://en.wikipedia.org/wiki/Briton_C._Busch'},
     {'birth_year': u'1931',
      'gender': 'Male',
      'name': 'Richard Bushman',
      'url': 'https://en.wikipedia.org/wiki/Richard_Bushman'},
     {'birth_year': u'1900',
      'gender': 'Male',
      'name': 'Herbert Butterfield',
      'url': 'https://en.wikipedia.org/wiki/Herbert_Butterfield'},
     {'birth_year': u'1942',
      'gender': 'Male',
      'name': 'Angus Calder',
      'url': 'https://en.wikipedia.org/wiki/Angus_Calder'},
     {'birth_year': u'1914',
      'gender': 'Male',
      'name': 'Julio Caro Baroja',
      'url': 'https://en.wikipedia.org/wiki/Julio_Caro_Baroja'},
     {'birth_year': u'1919',
      'gender': 'Male',
      'name': 'Sir Raymond Carr',
      'url': 'https://en.wikipedia.org/wiki/Raymond_Carr'},
     {'birth_year': u'1947',
      'gender': 'Male',
      'name': 'Paul Cartledge',
      'url': 'https://en.wikipedia.org/wiki/Paul_Cartledge'},
     {'birth_year': u'1914',
      'gender': 'Male',
      'name': 'Lionel Casson',
      'url': 'https://en.wikipedia.org/wiki/Lionel_Casson'},
     {'birth_year': u'1923',
      'gender': 'Male',
      'name': 'Boris Celovsky',
      'url': 'https://en.wikipedia.org/wiki/Borivoj_Celovsky'},
     {'birth_year': u'1901',
      'gender': 'Male',
      'name': 'Howard I. Chapelle',
      'url': 'https://en.wikipedia.org/wiki/Howard_I._Chapelle'},
     {'birth_year': None,
      'gender': 'Male',
      'name': 'Maher Charif',
      'url': 'https://en.wikipedia.org/wiki/Maher_Charif'},
     {'birth_year': u'1968',
      'gender': 'Female',
      'name': 'Iris Chang',
      'url': 'https://en.wikipedia.org/wiki/Iris_Chang'},
     {'birth_year': u'1911',
      'gender': 'Male',
      'name': 'Louis Chevalier',
      'url': 'https://en.wikipedia.org/wiki/Louis_Chevalier'},
     {'birth_year': u'1976',
      'gender': 'Male',
      'name': 'Thomas Childers',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Childers'},
     {'birth_year': u'1919',
      'gender': 'Male',
      'name': 'I. R. Christie',
      'url': 'https://en.wikipedia.org/wiki/I._R._Christie'},
     {'birth_year': u'1924',
      'gender': '',
      'name': 'Alexander Campbell Cheyne',
      'url': 'https://en.wikipedia.org/wiki/Alexander_Campbell_Cheyne'},
     {'birth_year': '1935',
      'gender': 'Male',
      'name': 'Satyabrata Rai Chowdhuri',
      'url': 'https://en.wikipedia.org/wiki/Satyabrata_Rai_Chowdhuri'},
     {'birth_year': u'1928',
      'gender': 'Male',
      'name': 'Alan Clark',
      'url': 'https://en.wikipedia.org/wiki/Alan_Clark'},
     {'birth_year': u'1960',
      'gender': 'Male',
      'name': 'Christopher Clark',
      'url': 'https://en.wikipedia.org/wiki/Chris_Clark_(historian)'},
     {'birth_year': '1951',
      'gender': 'Male',
      'name': 'J.C.D. Clark',
      'url': 'https://en.wikipedia.org/wiki/J.C.D._Clark'},
     {'birth_year': u'1915',
      'gender': 'Male',
      'name': 'Manning Clark',
      'url': 'https://en.wikipedia.org/wiki/Manning_Clark'},
     {'birth_year': u'1929',
      'gender': 'Male',
      'name': 'Patrick Collinson',
      'url': 'https://en.wikipedia.org/wiki/Patrick_Collinson'},
     {'birth_year': u'1917',
      'gender': 'Male',
      'name': 'Robert Conquest',
      'url': 'https://en.wikipedia.org/wiki/Robert_Conquest'},
     {'birth_year': '1946',
      'gender': 'Female',
      'name': 'Margaret Conrad',
      'url': 'https://en.wikipedia.org/wiki/Margaret_Conrad'},
     {'birth_year': u'1885',
      'gender': 'Male',
      'name': u'Vladimir \u0106orovi\u0107',
      'url': 'https://en.wikipedia.org/wiki/Vladimir_%C4%86orovi%C4%87'},
     {'birth_year': u'1964',
      'gender': 'Male',
      'name': 'Peter Cottrell',
      'url': 'https://en.wikipedia.org/wiki/Peter_Cottrell'},
     {'birth_year': u'1913',
      'gender': 'Male',
      'name': 'Gordon A. Craig',
      'url': 'https://en.wikipedia.org/wiki/Gordon_A._Craig'},
     {'birth_year': u'1902',
      'gender': 'Male',
      'name': 'Donald Creighton',
      'url': 'https://en.wikipedia.org/wiki/Donald_Creighton'},
     {'birth_year': u'1924',
      'gender': 'Male',
      'name': 'Vincent Cronin',
      'url': 'https://en.wikipedia.org/wiki/Vincent_Cronin'},
     {'birth_year': u'1954',
      'gender': 'Male',
      'name': 'William Cronon',
      'url': 'https://en.wikipedia.org/wiki/William_Cronon'},
     {'birth_year': u'1955',
      'gender': 'Female',
      'name': 'Pamela Kyle Crossley',
      'url': 'https://en.wikipedia.org/wiki/Pamela_Kyle_Crossley'},
     {'birth_year': u'1949',
      'gender': 'Male',
      'name': 'Dan Cruickshank',
      'url': 'https://en.wikipedia.org/wiki/Dan_Cruickshank'},
     {'birth_year': u'1939',
      'gender': 'Male',
      'name': 'Barry Cunliffe',
      'url': 'https://en.wikipedia.org/wiki/Barry_Cunliffe'},
     {'birth_year': u'1899',
      'gender': 'Male',
      'name': 'John Shelton Curtiss',
      'url': 'https://en.wikipedia.org/wiki/John_Shelton_Curtiss'},
     {'birth_year': u'1934',
      'gender': 'Male',
      'name': 'Robert Dallek',
      'url': 'https://en.wikipedia.org/wiki/Robert_Dallek'},
     {'birth_year': u'1926',
      'gender': 'Male',
      'name': 'Vahakn N. Dadrian',
      'url': 'https://en.wikipedia.org/wiki/Vahakn_N._Dadrian'},
     {'birth_year': '1947',
      'gender': 'Male',
      'name': 'David B. Danbom',
      'url': 'https://en.wikipedia.org/wiki/David_B._Danbom'},
     {'birth_year': u'1920',
      'gender': 'Male',
      'name': 'Ahmad Hasan Dani',
      'url': 'https://en.wikipedia.org/wiki/Ahmad_Hasan_Dani'},
     {'birth_year': u'1939',
      'gender': 'Male',
      'name': 'Robert Darnton',
      'url': 'https://en.wikipedia.org/wiki/Robert_Darnton'},
     {'birth_year': u'1915',
      'gender': 'Female',
      'name': 'Lucy Dawidowicz',
      'url': 'https://en.wikipedia.org/wiki/Lucy_Dawidowicz'},
     {'birth_year': u'1966',
      'gender': 'Male',
      'name': 'Saul David',
      'url': 'https://en.wikipedia.org/wiki/Saul_David'},
     {'birth_year': u'1938',
      'gender': 'Male',
      'name': 'John Davies',
      'url': 'https://en.wikipedia.org/wiki/John_Davies_(historian)'},
     {'birth_year': u'1939',
      'gender': 'Male',
      'name': 'Norman Davies',
      'url': 'https://en.wikipedia.org/wiki/Norman_Davies'},
     {'birth_year': u'1928',
      'gender': 'Female',
      'name': 'Natalie Zemon Davis',
      'url': 'https://en.wikipedia.org/wiki/Natalie_Zemon_Davis'},
     {'birth_year': u'1912',
      'gender': 'Male',
      'name': 'Kenneth S. Davis',
      'url': 'https://en.wikipedia.org/wiki/Kenneth_S._Davis'},
     {'birth_year': u'1918',
      'gender': 'Male',
      'name': 'R.H.C. Davis',
      'url': 'https://en.wikipedia.org/wiki/Ralph_Henry_Carless_Davis'},
     {'birth_year': '1949',
      'gender': 'Male',
      'name': 'David Day',
      'url': 'https://en.wikipedia.org/wiki/David_Day_(historian)'},
     {'birth_year': u'1929',
      'gender': 'Male',
      'name': 'Renzo De Felice',
      'url': 'https://en.wikipedia.org/wiki/Renzo_De_Felice'},
     {'birth_year': u'1929',
      'gender': 'Male',
      'name': 'Len Deighton',
      'url': 'https://en.wikipedia.org/wiki/Len_Deighton'},
     {'birth_year': u'1921',
      'gender': 'Male',
      'name': 'Carl N. Degler',
      'url': 'https://en.wikipedia.org/wiki/Carl_N._Degler'},
     {'birth_year': u'1954',
      'gender': 'Female',
      'name': 'Esther Delisle',
      'url': 'https://en.wikipedia.org/wiki/Esther_Delisle'},
     {'birth_year': u'1923',
      'gender': 'Male',
      'name': 'Jean Delumeau',
      'url': 'https://en.wikipedia.org/wiki/Jean_Delumeau'},
     {'birth_year': u'1935',
      'gender': 'Male',
      'name': 'Marcel Detienne',
      'url': 'https://en.wikipedia.org/wiki/Marcel_Detienne'},
     {'birth_year': u'1903',
      'gender': 'Male',
      'name': 'Alexandre Deulofeu',
      'url': 'https://en.wikipedia.org/wiki/Alexandre_Deulofeu'},
     {'birth_year': u'1907',
      'gender': 'Male',
      'name': 'Isaac Deutscher',
      'url': 'https://en.wikipedia.org/wiki/Isaac_Deutscher'},
     {'birth_year': u'1945',
      'gender': 'Male',
      'name': 'Tom M. Devine',
      'url': 'https://en.wikipedia.org/wiki/Tom_M._Devine'},
     {'birth_year': u'1951',
      'gender': 'Male',
      'name': 'Wu Di',
      'url': 'https://en.wikipedia.org/wiki/Wu_Di_(film_critic_and_historian)'},
     {'birth_year': u'1915',
      'gender': 'Male',
      'name': 'Igor M. Diakonov',
      'url': 'https://en.wikipedia.org/wiki/Igor_M._Diakonov'},
     {'birth_year': u'1920',
      'gender': 'Male',
      'name': 'David Herbert Donald',
      'url': 'https://en.wikipedia.org/wiki/David_Herbert_Donald'},
     {'birth_year': u'1913',
      'gender': 'Male',
      'name': 'Gordon Donaldson',
      'url': 'https://en.wikipedia.org/wiki/Gordon_Donaldson'},
     {'birth_year': '1558',
      'gender': 'Female',
      'name': 'Susan Doran',
      'url': 'https://en.wikipedia.org/wiki/Susan_Doran'},
     {'birth_year': '1942',
      'gender': '',
      'name': 'William Doyle',
      'url': 'https://en.wikipedia.org/wiki/William_Doyle_(historian)'},
     {'birth_year': u'1919',
      'gender': 'Male',
      'name': 'Georges Duby',
      'url': 'https://en.wikipedia.org/wiki/Georges_Duby'},
     {'birth_year': u'1936',
      'gender': 'Male',
      'name': 'William S. Dudley',
      'url': 'https://en.wikipedia.org/wiki/William_S._Dudley'},
     {'birth_year': u'1909',
      'gender': 'Male',
      'name': 'Robert Dudley Edwards',
      'url': 'https://en.wikipedia.org/wiki/Robert_Dudley_Edwards'},
     {'birth_year': '1947',
      'gender': 'Male',
      'name': 'Eamon Duffy',
      'url': 'https://en.wikipedia.org/wiki/Eamon_Duffy'},
     {'birth_year': u'1921',
      'gender': 'Male',
      'name': 'A. Hunter Dupree',
      'url': 'https://en.wikipedia.org/wiki/A._Hunter_Dupree'},
     {'birth_year': u'1916',
      'gender': 'Male',
      'name': 'Trevor Dupuy',
      'url': 'https://en.wikipedia.org/wiki/Trevor_Dupuy'},
     {'birth_year': u'1917',
      'gender': 'Male',
      'name': 'Jean-Baptiste Duroselle',
      'url': 'https://en.wikipedia.org/wiki/Jean-Baptiste_Duroselle'},
     {'birth_year': u'1921',
      'gender': 'Male',
      'name': 'Harold James Dyos',
      'url': 'https://en.wikipedia.org/wiki/Harold_James_Dyos'},
     {'birth_year': u'1923',
      'gender': 'Female',
      'name': 'Elizabeth Eisenstein',
      'url': 'https://en.wikipedia.org/wiki/Elizabeth_Eisenstein'},
     {'birth_year': '1949',
      'gender': 'Male',
      'name': 'Geoff Eley',
      'url': 'https://en.wikipedia.org/wiki/Geoff_Eley'},
     {'birth_year': u'1930',
      'gender': 'Male',
      'name': 'John Elliott',
      'url': 'https://en.wikipedia.org/wiki/John_Elliott_(historian)'},
     {'birth_year': '1943',
      'gender': 'Male',
      'name': 'Joseph J. Ellis',
      'url': 'https://en.wikipedia.org/wiki/Joseph_J._Ellis'},
     {'birth_year': u'1921',
      'gender': 'Male',
      'name': 'Geoffrey Elton',
      'url': 'https://en.wikipedia.org/wiki/Geoffrey_Elton'},
     {'birth_year': u'1957',
      'gender': 'Male',
      'name': 'Peter Englund',
      'url': 'https://en.wikipedia.org/wiki/Peter_Englund'},
     {'birth_year': '1939',
      'gender': 'Male',
      'name': 'Robert Malcolm Errington',
      'url': 'https://en.wikipedia.org/wiki/Robert_Malcolm_Errington'},
     {'birth_year': u'1947',
      'gender': 'Male',
      'name': 'Richard J. Evans',
      'url': 'https://en.wikipedia.org/wiki/Richard_J._Evans'},
     {'birth_year': u'1905',
      'gender': 'Male',
      'name': 'Alf Evers',
      'url': 'https://en.wikipedia.org/wiki/Alf_Evers'},
     {'birth_year': u'1929',
      'gender': 'Male',
      'name': 'Brian Farrell',
      'url': 'https://en.wikipedia.org/wiki/Brian_Farrell_(broadcaster)'},
     {'birth_year': u'1918',
      'gender': 'Male',
      'name': 'John Lister Illingworth Fennell',
      'url': 'https://en.wikipedia.org/wiki/John_Lister_Illingworth_Fennell'},
     {'birth_year': u'1964',
      'gender': 'Male',
      'name': 'Niall Ferguson',
      'url': 'https://en.wikipedia.org/wiki/Niall_Ferguson'},
     {'birth_year': u'1929',
      'gender': 'Male',
      'name': u'Bo\u017eidar Ferjan\u010di\u0107',
      'url': 'https://en.wikipedia.org/wiki/Bo%C5%BEidar_Ferjan%C4%8Di%C4%87'},
     {'birth_year': u'1924',
      'gender': 'Male',
      'name': 'Marc Ferro',
      'url': 'https://en.wikipedia.org/wiki/Marc_Ferro'},
     {'birth_year': u'1926',
      'gender': 'Male',
      'name': 'Joachim Fest',
      'url': 'https://en.wikipedia.org/wiki/Joachim_Fest'},
     {'birth_year': u'1912',
      'gender': 'Male',
      'name': 'David Feuerwerker',
      'url': 'https://en.wikipedia.org/wiki/David_Feuerwerker'},
     {'birth_year': u'1912',
      'gender': 'Male',
      'name': 'Heinrich Fichtenau',
      'url': 'https://en.wikipedia.org/wiki/Heinrich_Fichtenau'},
     {'birth_year': '1925',
      'gender': 'Male',
      'name': 'David Kenneth Fieldhouse',
      'url': 'https://en.wikipedia.org/wiki/David_Kenneth_Fieldhouse'},
     {'birth_year': u'1959',
      'gender': 'Male',
      'name': 'Orlando Figes',
      'url': 'https://en.wikipedia.org/wiki/Orlando_Figes'},
     {'birth_year': u'1905',
      'gender': 'Male',
      'name': 'Robert O. Fink',
      'url': 'https://en.wikipedia.org/wiki/Robert_O._Fink'},
     {'birth_year': u'1912',
      'gender': 'Male',
      'name': 'Moses Finley',
      'url': 'https://en.wikipedia.org/wiki/Moses_Finley'},
     {'birth_year': u'1935',
      'gender': 'Male',
      'name': 'David Hackett Fischer',
      'url': 'https://en.wikipedia.org/wiki/David_Hackett_Fischer'},
     {'birth_year': u'1908',
      'gender': 'Male',
      'name': 'Fritz Fischer',
      'url': 'https://en.wikipedia.org/wiki/Fritz_Fischer_(historian)'},
     {'birth_year': '1940',
      'gender': 'Female',
      'name': 'Frances FitzGerald',
      'url': 'https://en.wikipedia.org/wiki/Frances_FitzGerald_(journalist)'},
     {'birth_year': '1959',
      'gender': 'Female',
      'name': 'Judith Flanders',
      'url': 'https://en.wikipedia.org/wiki/Judith_Flanders'},
     {'birth_year': u'1926',
      'gender': 'Male',
      'name': 'Robert Fogel',
      'url': 'https://en.wikipedia.org/wiki/Robert_Fogel'},
     {'birth_year': u'1943',
      'gender': 'Male',
      'name': 'Eric Foner',
      'url': 'https://en.wikipedia.org/wiki/Eric_Foner'},
     {'birth_year': u'1916',
      'gender': 'Male',
      'name': 'Shelby Foote',
      'url': 'https://en.wikipedia.org/wiki/Shelby_Foote'},
     {'birth_year': u'1968',
      'gender': 'Female',
      'name': 'Amanda Foreman',
      'url': 'https://en.wikipedia.org/wiki/Amanda_Foreman_(historian)'},
     {'birth_year': u'1926',
      'gender': 'Male',
      'name': 'Michel Foucault',
      'url': 'https://en.wikipedia.org/wiki/Michel_Foucault'},
     {'birth_year': '2007',
      'gender': 'Female',
      'name': 'Jo Fox',
      'url': 'https://en.wikipedia.org/wiki/Jo_Fox'},
     {'birth_year': '1946',
      'gender': 'Male',
      'name': 'Robin Lane Fox',
      'url': 'https://en.wikipedia.org/wiki/Robin_Lane_Fox'},
     {'birth_year': u'1938',
      'gender': 'Male',
      'name': 'Stephen Fox',
      'url': 'https://en.wikipedia.org/wiki/Stephen_Fox_(author/educator)'},
     {'birth_year': u'1941',
      'gender': 'Female',
      'name': 'Elizabeth Fox-Genovese',
      'url': 'https://en.wikipedia.org/wiki/Elizabeth_Fox-Genovese'},
     {'birth_year': u'1905',
      'gender': 'Male',
      'name': 'Walter Frank',
      'url': 'https://en.wikipedia.org/wiki/Walter_Frank'},
     {'birth_year': u'1934',
      'gender': 'Male',
      'name': 'H. Bruce Franklin',
      'url': 'https://en.wikipedia.org/wiki/H._Bruce_Franklin'},
     {'birth_year': u'1932',
      'gender': 'Female',
      'name': 'Antonia Fraser',
      'url': 'https://en.wikipedia.org/wiki/Antonia_Fraser'},
     {'birth_year': u'1916',
      'gender': 'Male',
      'name': 'Frank Freidel',
      'url': 'https://en.wikipedia.org/wiki/Frank_Freidel'},
     {'birth_year': u'1922',
      'gender': '',
      'name': 'Joseph Friedenson',
      'url': 'https://en.wikipedia.org/wiki/Joseph_Friedenson'},
     {'birth_year': None,
      'gender': '',
      'name': 'Henry Friedlander',
      'url': 'https://en.wikipedia.org/wiki/Henry_Friedlander'},
     {'birth_year': u'1932',
      'gender': 'Male',
      'name': u'Saul Friedl\xe4nder',
      'url': 'https://en.wikipedia.org/wiki/Saul_Friedl%C3%A4nder'},
     {'birth_year': u'1916',
      'gender': 'Male',
      'name': 'Sheppard Frere',
      'url': 'https://en.wikipedia.org/wiki/Sheppard_Frere'},
     {'birth_year': '1932',
      'gender': 'Male',
      'name': 'David Fromkin',
      'url': 'https://en.wikipedia.org/wiki/David_Fromkin'},
     {'birth_year': '1968',
      'gender': '',
      'name': 'Bruno Fuligni',
      'url': 'https://en.wikipedia.org/wiki/Bruno_Fuligni'},
     {'birth_year': u'1952',
      'gender': 'Male',
      'name': 'Francis Fukuyama',
      'url': 'https://en.wikipedia.org/wiki/Francis_Fukuyama'},
     {'birth_year': u'1927',
      'gender': 'Male',
      'name': u'Fran\xe7ois Furet',
      'url': 'https://en.wikipedia.org/wiki/Fran%C3%A7ois_Furet'},
     {'birth_year': u'1945',
      'gender': 'Male',
      'name': 'Femme Gaastra',
      'url': 'https://en.wikipedia.org/wiki/Femme_Gaastra'},
     {'birth_year': u'1941',
      'gender': 'Male',
      'name': 'John Lewis Gaddis',
      'url': 'https://en.wikipedia.org/wiki/John_Lewis_Gaddis'},
     {'birth_year': '1934',
      'gender': 'Male',
      'name': 'Lloyd Gardner',
      'url': 'https://en.wikipedia.org/wiki/Lloyd_Gardner'},
     {'birth_year': u'1923',
      'gender': 'Male',
      'name': 'Peter Gay',
      'url': 'https://en.wikipedia.org/wiki/Peter_Gay'},
     {'birth_year': u'1930',
      'gender': 'Male',
      'name': 'Eugene Genovese',
      'url': 'https://en.wikipedia.org/wiki/Eugene_Genovese'},
     {'birth_year': u'2005',
      'gender': 'Male',
      'name': u'Fran\xe7ois G\xe9r\xe9',
      'url': 'https://en.wikipedia.org/wiki/Fran%C3%A7ois_G%C3%A9r%C3%A9'},
     {'birth_year': u'1931',
      'gender': 'Female',
      'name': 'Imanuel Geiss',
      'url': 'https://en.wikipedia.org/wiki/Imanuel_Geiss'},
     {'birth_year': u'1998',
      'gender': 'Male',
      'name': 'Christian Gerlach',
      'url': 'https://en.wikipedia.org/wiki/Christian_Gerlach'},
     {'birth_year': u'1910',
      'gender': 'Male',
      'name': 'N.H. Gibbs',
      'url': 'https://en.wikipedia.org/wiki/N.H._Gibbs'},
     {'birth_year': u'1959',
      'gender': 'Male',
      'name': 'William Gibson',
      'url': 'https://en.wikipedia.org/wiki/William_Gibson_(historian)'},
     {'birth_year': u'1936',
      'gender': 'Male',
      'name': 'Martin Gilbert',
      'url': 'https://en.wikipedia.org/wiki/Martin_Gilbert'},
     {'birth_year': u'1939',
      'gender': 'Male',
      'name': 'Carlo Ginzburg',
      'url': 'https://en.wikipedia.org/wiki/Carlo_Ginzburg'},
     {'birth_year': u'1947',
      'gender': 'Male',
      'name': 'Jan Glete',
      'url': 'https://en.wikipedia.org/wiki/Jan_Glete'},
     {'birth_year': u'1916',
      'gender': 'Male',
      'name': 'Eric F. Goldman',
      'url': 'https://en.wikipedia.org/wiki/Eric_F._Goldman'},
     {'birth_year': u'1958',
      'gender': 'Male',
      'name': 'James Goldrick',
      'url': 'https://en.wikipedia.org/wiki/James_Goldrick'},
     {'birth_year': u'1969',
      'gender': 'Male',
      'name': 'Adrian Goldsworthy',
      'url': 'https://en.wikipedia.org/wiki/Adrian_Goldsworthy'},
     {'birth_year': u'1925',
      'gender': 'Male',
      'name': 'Brison D. Gooch',
      'url': 'https://en.wikipedia.org/wiki/Brison_D._Gooch'},
     {'birth_year': u'1943',
      'gender': 'Female',
      'name': 'Doris Kearns Goodwin',
      'url': 'https://en.wikipedia.org/wiki/Doris_Kearns_Goodwin'},
     {'birth_year': '1951',
      'gender': 'Male',
      'name': 'Andrew Gordon',
      'url': 'https://en.wikipedia.org/wiki/Andrew_Gordon_(naval_historian)'},
     {'birth_year': u'1903',
      'gender': 'Male',
      'name': 'Gerald S. Graham',
      'url': 'https://en.wikipedia.org/wiki/Gerald_S._Graham'},
     {'birth_year': '1939',
      'gender': 'Male',
      'name': 'Jack Granatstein',
      'url': 'https://en.wikipedia.org/wiki/Jack_Granatstein'},
     {'birth_year': '1924',
      'gender': 'Male',
      'name': 'Peter Green',
      'url': 'https://en.wikipedia.org/wiki/Peter_Green_(historian)'},
     {'birth_year': u'1915',
      'gender': 'Male',
      'name': 'Vivian H.H. Green',
      'url': 'https://en.wikipedia.org/wiki/Rev._Vivian_Green'},
     {'birth_year': '1955',
      'gender': 'Male',
      'name': 'John Robert Greene',
      'url': 'https://en.wikipedia.org/wiki/John_Robert_Greene'},
     {'birth_year': '1948',
      'gender': 'Male',
      'name': 'Roger D. Griffin',
      'url': 'https://en.wikipedia.org/wiki/Roger_D._Griffin'},
     {'birth_year': u'1923',
      'gender': 'Male',
      'name': 'Ranajit Guha',
      'url': 'https://en.wikipedia.org/wiki/Ranajit_Guha'},
     {'birth_year': u'1958',
      'gender': 'Male',
      'name': 'Ramchandra Guha',
      'url': 'https://en.wikipedia.org/wiki/Ramchandra_Guha'},
     {'birth_year': u'1912',
      'gender': 'Male',
      'name': 'Lev Gumilyov',
      'url': 'https://en.wikipedia.org/wiki/Lev_Gumilyov'},
     {'birth_year': u'1911',
      'gender': 'Male',
      'name': 'Oliver Gurney',
      'url': 'https://en.wikipedia.org/wiki/Oliver_Gurney'},
     {'birth_year': '1949',
      'gender': 'Male',
      'name': 'John Guy',
      'url': 'https://en.wikipedia.org/wiki/John_Guy_(historian)'},
     {'birth_year': u'1931',
      'gender': 'Male',
      'name': 'Irfan Habib',
      'url': 'https://en.wikipedia.org/wiki/Irfan_Habib'},
     {'birth_year': u'1933',
      'gender': 'Male',
      'name': 'Sheldon Hackney',
      'url': 'https://en.wikipedia.org/wiki/Sheldon_Hackney'},
     {'birth_year': u'1936',
      'gender': 'Male',
      'name': 'Kenneth J. Hagan',
      'url': 'https://en.wikipedia.org/wiki/Kenneth_J._Hagan'},
     {'birth_year': u'1922',
      'gender': 'Male',
      'name': 'Claude Hall',
      'url': 'https://en.wikipedia.org/wiki/Claude_Hall'},
     {'birth_year': u'1916',
      'gender': 'Male',
      'name': 'John Whitney Hall',
      'url': 'https://en.wikipedia.org/wiki/John_Whitney_Hall'},
     {'birth_year': u'1937',
      'gender': 'Male',
      'name': 'Bruce Barrymore Halpenny',
      'url': 'https://en.wikipedia.org/wiki/Bruce_Barrymore_Halpenny'},
     {'birth_year': u'1907',
      'gender': 'Male',
      'name': 'N. G. L. Hammond',
      'url': 'https://en.wikipedia.org/wiki/N._G._L._Hammond'},
     {'birth_year': u'1953',
      'gender': 'Male',
      'name': 'Victor Davis Hanson',
      'url': 'https://en.wikipedia.org/wiki/Victor_Davis_Hanson'},
     {'birth_year': u'1948',
      'gender': 'Male',
      'name': 'Syed Nomanul Haq',
      'url': 'https://en.wikipedia.org/wiki/Syed_Nomanul_Haq'},
     {'birth_year': u'1966',
      'gender': 'Male',
      'name': 'Dick Harrison',
      'url': 'https://en.wikipedia.org/wiki/Dick_Harrison'},
     {'birth_year': u'1955',
      'gender': 'Male',
      'name': 'Peter Harrison',
      'url': 'https://en.wikipedia.org/wiki/Peter_Harrison_(historian)'},
     {'birth_year': u'1945',
      'gender': 'Male',
      'name': 'Max Hastings',
      'url': 'https://en.wikipedia.org/wiki/Max_Hastings'},
     {'birth_year': u'1941',
      'gender': 'Male',
      'name': 'John Hattendorf',
      'url': 'https://en.wikipedia.org/wiki/John_Hattendorf'},
     {'birth_year': u'1913',
      'gender': 'Female',
      'name': 'Ragnhild Hatton',
      'url': 'https://en.wikipedia.org/wiki/Ragnhild_Hatton'},
     {'birth_year': u'1915',
      'gender': 'Male',
      'name': 'Denys Hay',
      'url': 'https://en.wikipedia.org/wiki/Denys_Hay'},
     {'birth_year': u'1902',
      'gender': 'Male',
      'name': 'John Daniel Hayes',
      'url': 'https://en.wikipedia.org/wiki/John_Daniel_Hayes'},
     {'birth_year': u'1968',
      'gender': 'Male',
      'name': 'Ingo Heidbrink',
      'url': 'https://en.wikipedia.org/wiki/Ingo_Heidbrink'},
     {'birth_year': u'1947',
      'gender': 'Male',
      'name': 'Jeffrey Herf',
      'url': 'https://en.wikipedia.org/wiki/Jeffrey_Herf'},
     {'birth_year': '1956',
      'gender': 'Male',
      'name': 'Arthur Herman',
      'url': 'https://en.wikipedia.org/wiki/Arthur_Herman'},
     {'birth_year': u'1948',
      'gender': 'Male',
      'name': 'Michael Hicks',
      'url': 'https://en.wikipedia.org/wiki/Michael_Hicks_(historian)'},
     {'birth_year': u'1926',
      'gender': 'Male',
      'name': 'Raul Hilberg',
      'url': 'https://en.wikipedia.org/wiki/Raul_Hilberg'},
     {'birth_year': u'1941',
      'gender': 'Male',
      'name': 'Klaus Hildebrand',
      'url': 'https://en.wikipedia.org/wiki/Klaus_Hildebrand'},
     {'birth_year': u'1912',
      'gender': 'Male',
      'name': 'Christopher Hill',
      'url': 'https://en.wikipedia.org/wiki/Christopher_Hill_(historian)'},
     {'birth_year': u'1925',
      'gender': 'Male',
      'name': 'Andreas Hillgruber',
      'url': 'https://en.wikipedia.org/wiki/Andreas_Hillgruber'},
     {'birth_year': u'1936',
      'gender': 'Male',
      'name': 'Richard L. Hills',
      'url': 'https://en.wikipedia.org/wiki/Richard_L._Hills'},
     {'birth_year': u'1922',
      'gender': 'Female',
      'name': 'Gertrude Himmelfarb',
      'url': 'https://en.wikipedia.org/wiki/Gertrude_Himmelfarb'},
     {'birth_year': u'1918',
      'gender': 'Male',
      'name': 'Harry Hinsley',
      'url': 'https://en.wikipedia.org/wiki/Harry_Hinsley'},
     {'birth_year': u'1917',
      'gender': 'Male',
      'name': 'Eric Hobsbawm',
      'url': 'https://en.wikipedia.org/wiki/Eric_Hobsbawm'},
     {'birth_year': u'1922',
      'gender': 'Male',
      'name': 'Marshall Hodgson',
      'url': 'https://en.wikipedia.org/wiki/Marshall_Hodgson'},
     {'birth_year': None,
      'gender': '',
      'name': 'Richard Hofstadter',
      'url': 'https://en.wikipedia.org/wiki/Richard_Hofstadter'},
     {'birth_year': u'1930',
      'gender': 'Male',
      'name': 'Peter Hoffmann',
      'url': 'https://en.wikipedia.org/wiki/Peter_Hoffmann_(historian)'},
     {'birth_year': None,
      'gender': '',
      'name': 'David Hoggan',
      'url': 'https://en.wikipedia.org/wiki/David_Hoggan'},
     {'birth_year': u'1902',
      'gender': 'Male',
      'name': 'Hajo Holborn',
      'url': 'https://en.wikipedia.org/wiki/Hajo_Holborn'},
     {'birth_year': u'1968',
      'gender': 'Male',
      'name': 'Tom Holland',
      'url': 'https://en.wikipedia.org/wiki/Tom_Holland_(author)'},
     {'birth_year': u'1930',
      'gender': 'Male',
      'name': 'C. Warren Hollister',
      'url': 'https://en.wikipedia.org/wiki/C._Warren_Hollister'},
     {'birth_year': u'1927',
      'gender': 'Male',
      'name': 'George Holmes (professor)',
      'url': 'https://en.wikipedia.org/wiki/George_Holmes_(professor)'},
     {'birth_year': u'1946',
      'gender': 'Male',
      'name': 'Richard Holmes',
      'url': 'https://en.wikipedia.org/wiki/Richard_Holmes_(historian)'},
     {'birth_year': u'1964',
      'gender': 'Male',
      'name': 'Ed Hooper',
      'url': 'https://en.wikipedia.org/wiki/Ed_Hooper'},
     {'birth_year': u'1938',
      'gender': 'Male',
      'name': 'A.G. Hopkins',
      'url': 'https://en.wikipedia.org/wiki/A.G._Hopkins'},
     {'birth_year': u'1934',
      'gender': 'Male',
      'name': 'Keith Hopkins',
      'url': 'https://en.wikipedia.org/wiki/Keith_hopkins'},
     {'birth_year': u'1915',
      'gender': 'Male',
      'name': 'Albert Hourani',
      'url': 'https://en.wikipedia.org/wiki/Albert_Hourani'},
     {'birth_year': u'1931',
      'gender': 'Male',
      'name': 'Youssef Hourany',
      'url': 'https://en.wikipedia.org/wiki/Youssef_Hourany'},
     {'birth_year': u'1954',
      'gender': 'Male',
      'name': 'Daniel Horowitz',
      'url': 'https://en.wikipedia.org/wiki/Daniel_Horowitz'},
     {'birth_year': '1942',
      'gender': 'Female',
      'name': 'Helen Lefkowitz Horowitz',
      'url': 'https://en.wikipedia.org/wiki/Helen_Lefkowitz_Horowitz'},
     {'birth_year': u'1939',
      'gender': 'Male',
      'name': 'Michiel Horn',
      'url': 'https://en.wikipedia.org/wiki/Michiel_Horn'},
     {'birth_year': u'1925',
      'gender': 'Male',
      'name': 'Alistair Horne',
      'url': 'https://en.wikipedia.org/wiki/Alistair_Horne'},
     {'birth_year': u'1922',
      'gender': 'Male',
      'name': 'Michael Howard',
      'url': 'https://en.wikipedia.org/wiki/Michael_Howard_(historian)'},
     {'birth_year': None,
      'gender': '',
      'name': 'Robert Hughes',
      'url': 'https://en.wikipedia.org/wiki/Robert_Hughes_(critic)'},
     {'birth_year': '1968',
      'gender': 'Male',
      'name': 'Andrew Hunt',
      'url': 'https://en.wikipedia.org/wiki/Andrew_Hunt_(historian)'},
     {'birth_year': u'1974',
      'gender': 'Male',
      'name': 'Tristram Hunt',
      'url': 'https://en.wikipedia.org/wiki/Tristram_Hunt'},
     {'birth_year': u'1974',
      'gender': 'Male',
      'name': 'Mark C. Hunter',
      'url': 'https://en.wikipedia.org/wiki/Mark_C._Hunter'},
     {'birth_year': u'1916',
      'gender': 'Male',
      'name': 'Halil Inalcik',
      'url': 'https://en.wikipedia.org/wiki/Halil_Inalcik'},
     {'birth_year': '1946',
      'gender': 'Male',
      'name': 'Iqbal, Sheikh Mohammad',
      'url': 'https://en.wikipedia.org/wiki/Sheikh_Mohammad_Iqbal'},
     {'birth_year': u'1946',
      'gender': 'Male',
      'name': 'Jonathan Israel',
      'url': 'https://en.wikipedia.org/wiki/Jonathan_Israel'},
     {'birth_year': u'1929',
      'gender': 'Male',
      'name': u'Eberhard J\xe4ckel',
      'url': 'https://en.wikipedia.org/wiki/Eberhard_J%C3%A4ckel'},
     {'birth_year': u'1954',
      'gender': 'Male',
      'name': 'Julian T. Jackson',
      'url': 'https://en.wikipedia.org/wiki/Julian_T._Jackson'},
     {'birth_year': u'1956',
      'gender': 'Male',
      'name': 'Harold James',
      'url': 'https://en.wikipedia.org/wiki/Harold_James_(historian)'},
     {'birth_year': u'1931',
      'gender': 'Male',
      'name': 'Nikoloz Janashia',
      'url': 'https://en.wikipedia.org/wiki/Nikoloz_Janashia'},
     {'birth_year': u'1900',
      'gender': 'Male',
      'name': 'Simon Janashia',
      'url': 'https://en.wikipedia.org/wiki/Simon_Janashia'},
     {'birth_year': u'1922',
      'gender': '',
      'name': 'Marius Jansen',
      'url': 'https://en.wikipedia.org/wiki/Marius_Jansen'},
     {'birth_year': u'1909',
      'gender': 'Male',
      'name': 'Pawel Jasienica',
      'url': 'https://en.wikipedia.org/wiki/Pawel_Jasienica'},
     {'birth_year': u'1905',
      'gender': 'Male',
      'name': 'Merrill Jensen',
      'url': 'https://en.wikipedia.org/wiki/Merrill_Jensen_(historian)'},
     {'birth_year': u'1941',
      'gender': 'Male',
      'name': 'Richard J. Jensen',
      'url': 'https://en.wikipedia.org/wiki/Richard_J._Jensen'},
     {'birth_year': '1973',
      'gender': '',
      'name': 'Khasnor Johan',
      'url': 'https://en.wikipedia.org/wiki/Khasnor_Johan'},
     {'birth_year': u'1928',
      'gender': 'Male',
      'name': 'Paul Johnson',
      'url': 'https://en.wikipedia.org/wiki/Paul_Johnson_(writer)'},
     {'birth_year': u'1923',
      'gender': 'Male',
      'name': 'Robert Erwin Johnson',
      'url': 'https://en.wikipedia.org/wiki/Robert_Erwin_Johnson'},
     {'birth_year': u'1924',
      'gender': 'Male',
      'name': 'Mauno Jokipii',
      'url': 'https://en.wikipedia.org/wiki/Mauno_Jokipii'},
     {'birth_year': u'1904',
      'gender': 'Male',
      'name': 'A.H.M. Jones',
      'url': 'https://en.wikipedia.org/wiki/Arnold_Hugh_Martin_Jones'},
     {'birth_year': u'1907',
      'gender': 'Male',
      'name': 'Gwyn Jones',
      'url': 'https://en.wikipedia.org/wiki/Gwyn_Jones_(author)'},
     {'birth_year': u'1924',
      'gender': 'Male',
      'name': 'George Hilton Jones III',
      'url': 'https://en.wikipedia.org/wiki/George_Hilton_Jones,_III_(historian)'},
     {'birth_year': u'1914',
      'gender': 'Male',
      'name': 'Loe de Jong',
      'url': 'https://en.wikipedia.org/wiki/Loe_de_Jong'},
     {'birth_year': None,
      'gender': '',
      'name': 'Tony Judt',
      'url': 'https://en.wikipedia.org/wiki/Tony_Judt'},
     {'birth_year': '1953',
      'gender': 'Male',
      'name': 'David S. Katz',
      'url': 'https://en.wikipedia.org/wiki/David_S._Katz'},
     {'birth_year': u'1932',
      'gender': 'Male',
      'name': 'Donald Kagan',
      'url': 'https://en.wikipedia.org/wiki/Donald_Kagan'},
     {'birth_year': u'1926',
      'gender': 'Male',
      'name': 'Elie Kedourie',
      'url': 'https://en.wikipedia.org/wiki/Elie_Kedourie'},
     {'birth_year': u'1937',
      'gender': 'Male',
      'name': 'Rod Kedward',
      'url': 'https://en.wikipedia.org/wiki/Rod_Kedward'},
     {'birth_year': u'1934',
      'gender': 'Male',
      'name': 'John Keegan',
      'url': 'https://en.wikipedia.org/wiki/John_Keegan'},
     {'birth_year': u'1937',
      'gender': 'Male',
      'name': 'Nushiravan Keihanizadeh',
      'url': 'https://en.wikipedia.org/wiki/Nushiravan_Keihanizadeh'},
     {'birth_year': u'1912',
      'gender': 'Male',
      'name': 'John H. Kemble',
      'url': 'https://en.wikipedia.org/wiki/John_H._Kemble'},
     {'birth_year': u'1911',
      'gender': 'Male',
      'name': 'Paul Murray Kendall',
      'url': 'https://en.wikipedia.org/wiki/Paul_Murray_Kendall'},
     {'birth_year': u'1938',
      'gender': 'Female',
      'name': 'Elizabeth Topham Kennan',
      'url': 'https://en.wikipedia.org/wiki/Elizabeth_Topham_Kennan'},
     {'birth_year': u'1904',
      'gender': 'Male',
      'name': 'George F. Kennan',
      'url': 'https://en.wikipedia.org/wiki/George_F._Kennan'},
     {'birth_year': '1963',
      'gender': 'Male',
      'name': 'James Kennedy',
      'url': 'https://en.wikipedia.org/wiki/James_Kennedy_(historian)'},
     {'birth_year': u'1945',
      'gender': 'Male',
      'name': 'Paul Kennedy',
      'url': 'https://en.wikipedia.org/wiki/Paul_Kennedy'},
     {'birth_year': u'1928',
      'gender': 'Male',
      'name': 'W. Hudson Kensel',
      'url': 'https://en.wikipedia.org/wiki/W._Hudson_Kensel'},
     {'birth_year': u'1943',
      'gender': 'Male',
      'name': 'Ian Kershaw',
      'url': 'https://en.wikipedia.org/wiki/Ian_Kershaw'},
     {'birth_year': '1939',
      'gender': 'Male',
      'name': 'Daniel J. Kevles',
      'url': 'https://en.wikipedia.org/wiki/Daniel_J._Kevles'},
     {'birth_year': u'1914',
      'gender': 'Male',
      'name': 'Khan Roshan Khan',
      'url': 'https://en.wikipedia.org/wiki/Khan_Roshan_Khan'},
     {'birth_year': '1940',
      'gender': 'Male',
      'name': 'Kim Jung-bae',
      'url': 'https://en.wikipedia.org/wiki/Kim_Jung-bae'},
     {'birth_year': u'1945',
      'gender': 'Male',
      'name': 'Michael King',
      'url': 'https://en.wikipedia.org/wiki/Michael_King'},
     {'birth_year': u'1904',
      'gender': 'Male',
      'name': 'Patrick Kinross',
      'url': 'https://en.wikipedia.org/wiki/Patrick_Kinross'},
     {'birth_year': u'1923',
      'gender': 'Male',
      'name': 'Henry Kissinger',
      'url': 'https://en.wikipedia.org/wiki/Henry_Kissinger'},
     {'birth_year': '1936',
      'gender': 'Male',
      'name': 'Martin Kitchen',
      'url': 'https://en.wikipedia.org/wiki/Martin_Kitchen'},
     {'birth_year': u'1967',
      'gender': 'Male',
      'name': 'Simon Kitson',
      'url': 'https://en.wikipedia.org/wiki/Simon_Kitson'},
     {'birth_year': u'1936',
      'gender': 'Male',
      'name': 'Matti Klinge',
      'url': 'https://en.wikipedia.org/wiki/Matti_Klinge'},
     {'birth_year': u'1992',
      'gender': 'Male',
      'name': 'Felix Klos',
      'url': 'https://en.wikipedia.org/wiki/Felix_Klos'},
     {'birth_year': u'1944',
      'gender': 'Male',
      'name': 'R.J.B. Knight',
      'url': 'https://en.wikipedia.org/wiki/R.J.B._Knight'},
     {'birth_year': u'1922',
      'gender': 'Male',
      'name': 'Yuri Knorozov',
      'url': 'https://en.wikipedia.org/wiki/Yuri_Knorozov'},
     {'birth_year': u'1933',
      'gender': 'Male',
      'name': 'Eberhard Kolb',
      'url': 'https://en.wikipedia.org/wiki/Eberhard_Kolb'},
     {'birth_year': u'1932',
      'gender': 'Male',
      'name': 'Gabriel Kolko',
      'url': 'https://en.wikipedia.org/wiki/Gabriel_Kolko'},
     {'birth_year': '1940',
      'gender': 'Female',
      'name': 'Claudia Koonz',
      'url': 'https://en.wikipedia.org/wiki/Claudia_Koonz'},
     {'birth_year': u'1961',
      'gender': 'Male',
      'name': 'Andrey Korotayev',
      'url': 'https://en.wikipedia.org/wiki/Andrey_Korotayev'},
     {'birth_year': u'1922',
      'gender': 'Male',
      'name': 'Ernst Kossmann',
      'url': 'https://en.wikipedia.org/wiki/Ernst_Kossmann'},
     {'birth_year': u'1933',
      'gender': 'Male',
      'name': 'Philip A. Kuhn',
      'url': 'https://en.wikipedia.org/wiki/Philip_A._Kuhn'},
     {'birth_year': u'1922',
      'gender': 'Male',
      'name': 'Thomas Kuhn',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Kuhn'},
     {'birth_year': u'1960',
      'gender': 'Male',
      'name': 'Myoma Myint Kywe',
      'url': 'https://en.wikipedia.org/wiki/Myoma_Myint_Kywe'},
     {'birth_year': u'1920',
      'gender': 'Male',
      'name': 'K.S. Lal',
      'url': 'https://en.wikipedia.org/wiki/K.S._Lal'},
     {'birth_year': u'1927',
      'gender': 'Male',
      'name': 'Benjamin Woods Labaree',
      'url': 'https://en.wikipedia.org/wiki/Benjamin_Woods_Labaree'},
     {'birth_year': u'2006',
      'gender': 'Male',
      'name': 'Brij Lal',
      'url': 'https://en.wikipedia.org/wiki/Brij_Lal_(historian)'},
     {'birth_year': u'1933',
      'gender': 'Male',
      'name': 'Abdallah Laroui',
      'url': 'https://en.wikipedia.org/wiki/Abdallah_Laroui'},
     {'birth_year': u'1920',
      'gender': 'Male',
      'name': 'Leopold Labedz',
      'url': 'https://en.wikipedia.org/wiki/Leopold_Labedz'},
     {'birth_year': None,
      'gender': '',
      'name': 'Andrew Lambert',
      'url': 'https://en.wikipedia.org/wiki/Andrew_Lambert'},
     {'birth_year': u'1892',
      'gender': 'Male',
      'name': 'Harold Lamb',
      'url': 'https://en.wikipedia.org/wiki/Harold_Lamb'},
     {'birth_year': u'1905',
      'gender': 'Male',
      'name': 'Ricardo Lancaster-Jones y Verea',
      'url': 'https://en.wikipedia.org/wiki/Ricardo_Lancaster-Jones_y_Verea'},
     {'birth_year': u'1910',
      'gender': 'Male',
      'name': 'David Lavender',
      'url': 'https://en.wikipedia.org/wiki/David_Lavender'},
     {'birth_year': u'1933',
      'gender': 'Male',
      'name': 'Walter LaFeber',
      'url': 'https://en.wikipedia.org/wiki/Walter_LaFeber'},
     {'birth_year': u'1936',
      'gender': 'Male',
      'name': 'Daniel Leab',
      'url': 'https://en.wikipedia.org/wiki/Daniel_Leab'},
     {'birth_year': u'1924',
      'gender': 'Male',
      'name': 'Jacques Le Goff',
      'url': 'https://en.wikipedia.org/wiki/Jacques_Le_Goff'},
     {'birth_year': u'1920',
      'gender': 'Male',
      'name': 'Robert Leckie',
      'url': 'https://en.wikipedia.org/wiki/Robert_Leckie_(author)'},
     {'birth_year': '1922',
      'gender': 'Male',
      'name': 'William Leuchtenburg',
      'url': 'https://en.wikipedia.org/wiki/William_Leuchtenburg'},
     {'birth_year': '1931',
      'gender': 'Female',
      'name': 'Barbara Levick',
      'url': 'https://en.wikipedia.org/wiki/Barbara_Levick'},
     {'birth_year': '1936',
      'gender': 'Male',
      'name': 'David Levering Lewis',
      'url': 'https://en.wikipedia.org/wiki/David_Levering_Lewis'},
     {'birth_year': u'1929',
      'gender': 'Male',
      'name': 'Emmanuel Le Roy Ladurie',
      'url': 'https://en.wikipedia.org/wiki/Emmanuel_Le_Roy_Ladurie'},
     {'birth_year': u'1924',
      'gender': 'Male',
      'name': 'Lee Ki-baek',
      'url': 'https://en.wikipedia.org/wiki/Lee_Ki-baek'},
     {'birth_year': u'1935',
      'gender': 'Male',
      'name': 'Li Ao',
      'url': 'https://en.wikipedia.org/wiki/Li_Ao'},
     {'birth_year': u'1929',
      'gender': 'Male',
      'name': 'Leon F. Litwack',
      'url': 'https://en.wikipedia.org/wiki/Leon_F._Litwack'},
     {'birth_year': '1951',
      'gender': 'Female',
      'name': 'Xinru Liu',
      'url': 'https://en.wikipedia.org/wiki/Xinru_Liu'},
     {'birth_year': '1939',
      'gender': 'Male',
      'name': 'Mario Liverani',
      'url': 'https://en.wikipedia.org/wiki/Mario_Liverani'},
     {'birth_year': u'1949',
      'gender': 'Male',
      'name': u'Rado\u0161 Lju\u0161i\u0107',
      'url': 'https://en.wikipedia.org/wiki/Rado%C5%A1_Lju%C5%A1i%C4%87'},
     {'birth_year': u'1934',
      'gender': 'Male',
      'name': 'David Loades',
      'url': 'https://en.wikipedia.org/wiki/David_Loades'},
     {'birth_year': u'1942',
      'gender': 'Male',
      'name': 'James W. Loewen',
      'url': 'https://en.wikipedia.org/wiki/James_W._Loewen'},
     {'birth_year': u'1906',
      'gender': 'Female',
      'name': 'Elizabeth Longford',
      'url': 'https://en.wikipedia.org/wiki/Elizabeth_Pakenham,_Countess_of_Longford'},
     {'birth_year': u'1910',
      'gender': 'Male',
      'name': u'Erik L\xf6nnroth',
      'url': 'https://en.wikipedia.org/wiki/Erik_L%C3%B6nnroth'},
     {'birth_year': u'1917',
      'gender': 'Male',
      'name': 'Walter Lord',
      'url': 'https://en.wikipedia.org/wiki/Walter_Lord'},
     {'birth_year': u'1924',
      'gender': 'Male',
      'name': 'John Lukacs',
      'url': 'https://en.wikipedia.org/wiki/John_Lukacs'},
     {'birth_year': u'1922',
      'gender': 'Male',
      'name': 'Charles B. MacDonald',
      'url': 'https://en.wikipedia.org/wiki/Charles_B._MacDonald'},
     {'birth_year': u'1947',
      'gender': 'Male',
      'name': 'Stuart Macintyre',
      'url': 'https://en.wikipedia.org/wiki/Stuart_Macintyre'},
     {'birth_year': u'1927',
      'gender': 'Male',
      'name': 'Forrest McDonald',
      'url': 'https://en.wikipedia.org/wiki/Forrest_McDonald'},
     {'birth_year': u'1903',
      'gender': 'Male',
      'name': 'K. B. McFarlane',
      'url': 'https://en.wikipedia.org/wiki/K._B._McFarlane'},
     {'birth_year': u'1942',
      'gender': 'Male',
      'name': 'Ross McKibbin',
      'url': 'https://en.wikipedia.org/wiki/Ross_McKibbin'},
     {'birth_year': '1949',
      'gender': 'Female',
      'name': 'Rosamond McKitterick',
      'url': 'https://en.wikipedia.org/wiki/Rosamond_McKitterick'},
     {'birth_year': u'1943',
      'gender': 'Female',
      'name': 'Margaret MacMillan',
      'url': 'https://en.wikipedia.org/wiki/Margaret_MacMillan'},
     {'birth_year': u'1885',
      'gender': 'Male',
      'name': 'William Miller Macmillan',
      'url': 'https://en.wikipedia.org/wiki/William_Miller_Macmillan'},
     {'birth_year': '1928',
      'gender': 'Male',
      'name': 'Ramsay MacMullen',
      'url': 'https://en.wikipedia.org/wiki/Ramsay_MacMullen'},
     {'birth_year': u'1929',
      'gender': 'Male',
      'name': 'Magnus Magnusson',
      'url': 'https://en.wikipedia.org/wiki/Magnus_Magnusson'},
     {'birth_year': u'1924',
      'gender': 'Male',
      'name': 'Piers Mackesy',
      'url': 'https://en.wikipedia.org/wiki/Piers_Mackesy'},
     {'birth_year': u'1950',
      'gender': 'Male',
      'name': 'Leonard Maltin',
      'url': 'https://en.wikipedia.org/wiki/Leonard_Maltin'},
     {'birth_year': '1939',
      'gender': 'Male',
      'name': 'Charles S. Maier',
      'url': 'https://en.wikipedia.org/wiki/Charles_S._Maier'},
     {'birth_year': u'1930',
      'gender': 'Male',
      'name': 'Paul L. Maier',
      'url': 'https://en.wikipedia.org/wiki/Paul_L._Maier'},
     {'birth_year': u'1938',
      'gender': 'Female',
      'name': 'Pauline Maier',
      'url': 'https://en.wikipedia.org/wiki/Pauline_Maier'},
     {'birth_year': u'1922',
      'gender': 'Male',
      'name': 'William Manchester',
      'url': 'https://en.wikipedia.org/wiki/William_Manchester'},
     {'birth_year': '1947',
      'gender': 'Male',
      'name': 'Adel Manna',
      'url': 'https://en.wikipedia.org/wiki/Adel_Manna'},
     {'birth_year': u'1909',
      'gender': 'Male',
      'name': 'Golo Mann',
      'url': 'https://en.wikipedia.org/wiki/Golo_Mann'},
     {'birth_year': u'1941',
      'gender': 'Female',
      'name': 'Susan Mann',
      'url': 'https://en.wikipedia.org/wiki/Susan_Mann'},
     {'birth_year': u'1920',
      'gender': 'Male',
      'name': 'Robert Mann',
      'url': 'https://en.wikipedia.org/wiki/Robert_Mann'},
     {'birth_year': u'1951',
      'gender': 'Male',
      'name': 'Philip Mansel',
      'url': 'https://en.wikipedia.org/wiki/Philip_Mansel'},
     {'birth_year': u'1910',
      'gender': 'Male',
      'name': 'Arthur Marder',
      'url': 'https://en.wikipedia.org/wiki/Arthur_Marder'},
     {'birth_year': u'1940',
      'gender': 'Male',
      'name': 'Timothy Mason',
      'url': 'https://en.wikipedia.org/wiki/Timothy_Mason'},
     {'birth_year': u'1924',
      'gender': 'Male',
      'name': 'Henri-Jean Martin',
      'url': 'https://en.wikipedia.org/wiki/Henri-Jean_Martin'},
     {'birth_year': u'1922',
      'gender': 'Male',
      'name': 'Rev. F.X. Martin',
      'url': 'https://en.wikipedia.org/wiki/F.X._Martin'},
     {'birth_year': u'1941',
      'gender': 'Male',
      'name': 'Michael Marrus',
      'url': 'https://en.wikipedia.org/wiki/Michael_Marrus'},
     {'birth_year': u'1958',
      'gender': 'Male',
      'name': 'Mark Mazower',
      'url': 'https://en.wikipedia.org/wiki/Mark_Mazower'},
     {'birth_year': u'1933',
      'gender': 'Male',
      'name': 'David McCullough',
      'url': 'https://en.wikipedia.org/wiki/David_McCullough'},
     {'birth_year': '1930',
      'gender': 'Male',
      'name': 'William S. McFeely',
      'url': 'https://en.wikipedia.org/wiki/William_S._McFeely'},
     {'birth_year': u'1936',
      'gender': 'Male',
      'name': 'James M. McPherson',
      'url': 'https://en.wikipedia.org/wiki/James_M._McPherson'},
     {'birth_year': u'1917',
      'gender': 'Male',
      'name': 'William McNeill',
      'url': 'https://en.wikipedia.org/wiki/William_Hardy_McNeill'},
     {'birth_year': '1209',
      'gender': 'Male',
      'name': 'Laurence Marvin',
      'url': 'https://en.wikipedia.org/wiki/Laurence_Marvin'},
     {'birth_year': u'1900',
      'gender': 'Male',
      'name': 'Garrett Mattingly',
      'url': 'https://en.wikipedia.org/wiki/Garrett_Mattingly'},
     {'birth_year': u'1926',
      'gender': 'Male',
      'name': 'Arno J. Mayer',
      'url': 'https://en.wikipedia.org/wiki/Arno_J._Mayer'},
     {'birth_year': u'1946',
      'gender': 'Male',
      'name': 'Richard Maybury',
      'url': 'https://en.wikipedia.org/wiki/Richard_J._Maybury'},
     {'birth_year': u'1935',
      'gender': 'Male',
      'name': 'Neil McKendrick',
      'url': 'https://en.wikipedia.org/wiki/Neil_McKendrick'},
     {'birth_year': '1924',
      'gender': 'Male',
      'name': 'D. W. Meinig',
      'url': 'https://en.wikipedia.org/wiki/D._W._Meinig'},
     {'birth_year': u'1936',
      'gender': 'Male',
      'name': 'Evaldo Cabral de Mello',
      'url': 'https://en.wikipedia.org/wiki/Evaldo_Cabral_de_Mello'},
     {'birth_year': '1975',
      'gender': 'Male',
      'name': 'Russell Menard',
      'url': 'https://en.wikipedia.org/wiki/Russell_Menard'},
     {'birth_year': u'1910',
      'gender': 'Male',
      'name': 'Thomas C. Mendenhall',
      'url': 'https://en.wikipedia.org/wiki/Thomas_C._Mendenhall_(historian)'},
     {'birth_year': u'2013',
      'gender': 'Male',
      'name': 'Josef W. Meri',
      'url': 'https://en.wikipedia.org/wiki/Josef_W._Meri'},
     {'birth_year': '1941',
      'gender': 'Female',
      'name': 'Barbara Metcalf',
      'url': 'https://en.wikipedia.org/wiki/Barbara_Metcalf'},
     {'birth_year': u'1937',
      'gender': '',
      'name': u'Rade Mihalj\u010di\u0107',
      'url': 'https://en.wikipedia.org/wiki/Rade_Mihalj%C4%8Di%C4%87'},
     {'birth_year': u'1905',
      'gender': 'Male',
      'name': 'Perry Miller',
      'url': 'https://en.wikipedia.org/wiki/Perry_Miller'},
     {'birth_year': u'1966',
      'gender': 'Male',
      'name': 'Giles Milton',
      'url': 'https://en.wikipedia.org/wiki/Giles_Milton'},
     {'birth_year': u'1950',
      'gender': 'Female',
      'name': u'Zora Mintalov\xe1 \u2013 Zubercov\xe1',
      'url': 'https://en.wikipedia.org/wiki/Zora_Mintalov%C3%A1_%E2%80%93_Zubercov%C3%A1'},
     {'birth_year': u'1927',
      'gender': 'Male',
      'name': 'Yagutil Mishiev',
      'url': 'https://en.wikipedia.org/wiki/Yagutil_Mishiev'},
     {'birth_year': u'1930',
      'gender': 'Male',
      'name': 'Hans Mommsen',
      'url': 'https://en.wikipedia.org/wiki/Hans_Mommsen'},
     {'birth_year': u'1930',
      'gender': 'Male',
      'name': 'Wolfgang Mommsen',
      'url': 'https://en.wikipedia.org/wiki/Wolfgang_Mommsen'},
     {'birth_year': u'1965',
      'gender': 'Male',
      'name': 'Simon Sebag Montefiore',
      'url': 'https://en.wikipedia.org/wiki/Simon_Sebag_Montefiore'},
     {'birth_year': u'1907',
      'gender': 'Male',
      'name': 'Theodore William Moody',
      'url': 'https://en.wikipedia.org/wiki/Theodore_William_Moody'},
     {'birth_year': u'1916',
      'gender': 'Male',
      'name': 'Edmund Morgan',
      'url': 'https://en.wikipedia.org/wiki/Edmund_Morgan_(historian)'},
     {'birth_year': u'1934',
      'gender': 'Male',
      'name': 'Kenneth O. Morgan',
      'url': 'https://en.wikipedia.org/wiki/Kenneth_O._Morgan'},
     {'birth_year': u'1917',
      'gender': 'Male',
      'name': 'William J. Morgan (historian)',
      'url': 'https://en.wikipedia.org/wiki/William_J._Morgan_(historian)'},
     {'birth_year': u'1887',
      'gender': 'Male',
      'name': 'Samuel Eliot Morison',
      'url': 'https://en.wikipedia.org/wiki/Samuel_Eliot_Morison'},
     {'birth_year': u'1948',
      'gender': 'Male',
      'name': 'Benny Morris',
      'url': 'https://en.wikipedia.org/wiki/Benny_Morris'},
     {'birth_year': u'1967',
      'gender': 'Male',
      'name': 'Ian Mortimer',
      'url': 'https://en.wikipedia.org/wiki/Ian_Mortimer_(historian)'},
     {'birth_year': u'1908',
      'gender': 'Male',
      'name': 'W.L. Morton',
      'url': 'https://en.wikipedia.org/wiki/W.L._Morton'},
     {'birth_year': u'1918',
      'gender': 'Male',
      'name': 'George Mosse',
      'url': 'https://en.wikipedia.org/wiki/George_Mosse'},
     {'birth_year': u'1907',
      'gender': '',
      'name': 'Roland Mousnier',
      'url': 'https://en.wikipedia.org/wiki/Roland_Mousnier'},
     {'birth_year': u'1941',
      'gender': 'Male',
      'name': 'Mubarak Ali',
      'url': 'https://en.wikipedia.org/wiki/Mubarak_Ali'},
     {'birth_year': u'1900',
      'gender': 'Male',
      'name': 'Joseph Needham',
      'url': 'https://en.wikipedia.org/wiki/Joseph_Needham'},
     {'birth_year': None,
      'gender': '',
      'name': 'Cynthia Neville',
      'url': 'https://en.wikipedia.org/wiki/Cynthia_Neville'},
     {'birth_year': None,
      'gender': '',
      'name': 'Leo Niehorster',
      'url': 'https://en.wikipedia.org/wiki/Leo_Niehorster'},
     {'birth_year': u'1927',
      'gender': 'Male',
      'name': 'Thomas Nipperdey',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Nipperdey'},
     {'birth_year': u'1923',
      'gender': 'Male',
      'name': 'Ernst Nolte',
      'url': 'https://en.wikipedia.org/wiki/Ernst_Nolte'},
     {'birth_year': u'1842',
      'gender': 'Male',
      'name': u'Stojan Novakovi\u0107',
      'url': 'https://en.wikipedia.org/wiki/Stojan_Novakovi%C4%87'},
     {'birth_year': u'1933',
      'gender': 'Male',
      'name': "Robin O'Neil",
      'url': 'https://en.wikipedia.org/wiki/Robin_O%27Neil'},
     {'birth_year': u'1975',
      'gender': 'Male',
      'name': 'Josiah Ober',
      'url': 'https://en.wikipedia.org/wiki/Josiah_Ober'},
     {'birth_year': u'1930',
      'gender': 'Male',
      'name': 'Heiko Oberman',
      'url': 'https://en.wikipedia.org/wiki/Heiko_Oberman'},
     {'birth_year': u'1925',
      'gender': 'Male',
      'name': 'W. H. Oliver',
      'url': 'https://en.wikipedia.org/wiki/W._H._Oliver'},
     {'birth_year': u'1955',
      'gender': 'Male',
      'name': 'Michael Oren',
      'url': 'https://en.wikipedia.org/wiki/Michael_Oren'},
     {'birth_year': u'1909',
      'gender': 'Female',
      'name': 'Margaret Ormsby',
      'url': 'https://en.wikipedia.org/wiki/Margaret_Ormsby'},
     {'birth_year': u'1947',
      'gender': 'Male',
      'name': u'\u0130lber Ortayl\u0131',
      'url': 'https://en.wikipedia.org/wiki/%C4%B0lber_Ortayl%C4%B1'},
     {'birth_year': u'1926',
      'gender': 'Male',
      'name': 'Fernand Ouellet',
      'url': 'https://en.wikipedia.org/wiki/Fernand_Ouellet'},
     {'birth_year': '1947',
      'gender': 'Male',
      'name': 'Richard Overy',
      'url': 'https://en.wikipedia.org/wiki/Richard_Overy'},
     {'birth_year': u'1939',
      'gender': 'Male',
      'name': 'Steven Ozment',
      'url': 'https://en.wikipedia.org/wiki/Steven_Ozment'},
     {'birth_year': '1933',
      'gender': 'Male',
      'name': 'Thomas Pakenham',
      'url': 'https://en.wikipedia.org/wiki/Thomas_Pakenham_(historian)'},
     {'birth_year': u'1947',
      'gender': 'Male',
      'name': 'Madhavan K. Palat',
      'url': 'https://en.wikipedia.org/wiki/Madhavan_K._Palat'},
     {'birth_year': u'1948',
      'gender': 'Male',
      'name': u'Hasan B\xfclent Paksoy',
      'url': 'https://en.wikipedia.org/wiki/Hasan_B%C3%BClent_Paksoy'},
     {'birth_year': u'1954',
      'gender': 'Male',
      'name': u'Ilan Papp\xe9',
      'url': 'https://en.wikipedia.org/wiki/Ilan_Papp%C3%A9'},
     {'birth_year': u'1943',
      'gender': 'Male',
      'name': 'Simo Parpola',
      'url': 'https://en.wikipedia.org/wiki/Simo_Parpola'},
     {'birth_year': u'1914',
      'gender': 'Male',
      'name': 'J. H. Parry',
      'url': 'https://en.wikipedia.org/wiki/J._H._Parry'},
     {'birth_year': '1909',
      'gender': 'Male',
      'name': 'T. T. Paterson',
      'url': 'https://en.wikipedia.org/wiki/T._T._Paterson'},
     {'birth_year': u'1940',
      'gender': 'Male',
      'name': 'Fred Patten',
      'url': 'https://en.wikipedia.org/wiki/Fred_Patten'},
     {'birth_year': u'1924',
      'gender': 'Male',
      'name': 'Peter Paret',
      'url': 'https://en.wikipedia.org/wiki/Peter_Paret'},
     {'birth_year': '1943',
      'gender': 'Male',
      'name': 'Geoffrey Parker',
      'url': 'https://en.wikipedia.org/wiki/Geoffrey_Parker_(historian)'},
     {'birth_year': u'1934',
      'gender': 'Male',
      'name': 'Stanley G. Payne',
      'url': 'https://en.wikipedia.org/wiki/Stanley_G._Payne'},
     {'birth_year': u'1921',
      'gender': 'Male',
      'name': 'Abel Paz',
      'url': 'https://en.wikipedia.org/wiki/Abel_Paz'},
     {'birth_year': u'1919',
      'gender': 'Male',
      'name': 'Morgan D. Peoples',
      'url': 'https://en.wikipedia.org/wiki/Morgan_D._Peoples'},
     {'birth_year': '1933',
      'gender': 'Male',
      'name': 'William Armstrong Percy',
      'url': 'https://en.wikipedia.org/wiki/William_Armstrong_Percy'},
     {'birth_year': u'1925',
      'gender': 'Male',
      'name': 'Bradford Perkins',
      'url': 'https://en.wikipedia.org/wiki/Bradford_Perkins_(historian)'},
     {'birth_year': u'1950',
      'gender': 'Male',
      'name': 'Detlev Peukert',
      'url': 'https://en.wikipedia.org/wiki/Detlev_Peukert'},
     {'birth_year': u'1927',
      'gender': 'Female',
      'name': 'Liza Picard',
      'url': 'https://en.wikipedia.org/wiki/Liza_Picard'},
     {'birth_year': '1949',
      'gender': 'Male',
      'name': 'David Pietrusza',
      'url': 'https://en.wikipedia.org/wiki/David_Pietrusza'},
     {'birth_year': u'1908',
      'gender': 'Male',
      'name': 'Boris B. Piotrovsky',
      'url': 'https://en.wikipedia.org/wiki/Boris_B._Piotrovsky'},
     {'birth_year': u'1923',
      'gender': 'Male',
      'name': 'Richard Pipes',
      'url': 'https://en.wikipedia.org/wiki/Richard_Pipes'},
     {'birth_year': u'1911',
      'gender': 'Male',
      'name': 'J.H. Plumb',
      'url': 'https://en.wikipedia.org/wiki/J.H._Plumb'},
     {'birth_year': u'1924',
      'gender': 'Male',
      'name': 'J. G. A. Pocock',
      'url': 'https://en.wikipedia.org/wiki/J._G._A._Pocock'},
     {'birth_year': u'1949',
      'gender': 'Male',
      'name': 'Kwok Kin Poon',
      'url': 'https://en.wikipedia.org/wiki/Kwok_Kin_Poon'},
     {'birth_year': u'1941',
      'gender': 'Female',
      'name': 'Barbara Corrado Pope',
      'url': 'https://en.wikipedia.org/wiki/Barbara_Corrado_Pope'},
     {'birth_year': u'1946',
      'gender': 'Male',
      'name': 'Roy Porter',
      'url': 'https://en.wikipedia.org/wiki/Roy_Porter'},
     {'birth_year': u'1912',
      'gender': 'Male',
      'name': 'Norman Pounds',
      'url': 'https://en.wikipedia.org/wiki/Norman_Pounds'},
     {'birth_year': u'1910',
      'gender': 'Male',
      'name': 'Gordon W. Prange',
      'url': 'https://en.wikipedia.org/wiki/Gordon_W._Prange'},
     {'birth_year': u'1917',
      'gender': 'Male',
      'name': 'Joshua Prawer',
      'url': 'https://en.wikipedia.org/wiki/Joshua_Prawer'},
     {'birth_year': '1943',
      'gender': 'Male',
      'name': 'Michael Prestwich',
      'url': 'https://en.wikipedia.org/wiki/Michael_Prestwich'},
     {'birth_year': u'1945',
      'gender': 'Male',
      'name': 'Clement Alexander Price',
      'url': 'https://en.wikipedia.org/wiki/Clement_Alexander_Price'},
     {'birth_year': u'1921',
      'gender': 'Male',
      'name': 'Francis Paul Prucha',
      'url': 'https://en.wikipedia.org/wiki/Francis_Paul_Prucha'},
     {'birth_year': u'1942',
      'gender': 'Male',
      'name': 'Janko Prunk',
      'url': 'https://en.wikipedia.org/wiki/Janko_Prunk'},
     {'birth_year': u'1910',
      'gender': 'Male',
      'name': 'Carroll Quigley',
      'url': 'https://en.wikipedia.org/wiki/Carroll_Quigley'},
     {'birth_year': '1923',
      'gender': 'Male',
      'name': 'Marc Raeff',
      'url': 'https://en.wikipedia.org/wiki/Marc_Raeff'},
     {'birth_year': u'1939',
      'gender': 'Male',
      'name': 'Werner Rahn',
      'url': 'https://en.wikipedia.org/wiki/Werner_Rahn'},
     {'birth_year': u'1947',
      'gender': 'Male',
      'name': 'Jack N. Rakove',
      'url': 'https://en.wikipedia.org/wiki/Jack_N._Rakove'},
     {'birth_year': u'1956',
      'gender': 'Male',
      'name': u'\u0160erbo Rastoder',
      'url': 'https://en.wikipedia.org/wiki/%C5%A0erbo_Rastoder'},
     {'birth_year': u'1918',
      'gender': 'Male',
      'name': u'Ren\xe9 R\xe9mond',
      'url': 'https://en.wikipedia.org/wiki/Ren%C3%A9_R%C3%A9mond'},
     {'birth_year': u'1947',
      'gender': 'Male',
      'name': 'Timothy Reuter',
      'url': 'https://en.wikipedia.org/wiki/Timothy_Reuter'},
     {'birth_year': u'1938',
      'gender': 'Male',
      'name': 'Henry A. Reynolds',
      'url': 'https://en.wikipedia.org/wiki/Henry_A._Reynolds'},
     {'birth_year': u'1929',
      'gender': 'Female',
      'name': 'Susan Reynolds',
      'url': 'https://en.wikipedia.org/wiki/Susan_Reynolds'},
     {'birth_year': u'1937',
      'gender': 'Male',
      'name': 'Richard Rhodes',
      'url': 'https://en.wikipedia.org/wiki/Richard_Rhodes'},
     {'birth_year': u'1923',
      'gender': 'Male',
      'name': 'Nicholas V. Riasanovsky',
      'url': 'https://en.wikipedia.org/wiki/Nicholas_V._Riasanovsky'},
     {'birth_year': u'1871',
      'gender': 'Male',
      'name': 'Admiral Sir Herbert Richmond',
      'url': 'https://en.wikipedia.org/wiki/Herbert_Richmond'},
     {'birth_year': u'1938',
      'gender': 'Male',
      'name': 'Jonathan Riley-Smith',
      'url': 'https://en.wikipedia.org/wiki/Jonathan_Riley-Smith'},
     {'birth_year': u'1931',
      'gender': 'Male',
      'name': 'Blaze Ristovski',
      'url': 'https://en.wikipedia.org/wiki/Blaze_Ristovski'},
     {'birth_year': u'1925',
      'gender': 'Male',
      'name': 'Charles Ritcheson',
      'url': 'https://en.wikipedia.org/wiki/Charles_Ritcheson'},
     {'birth_year': u'1888',
      'gender': 'Male',
      'name': 'Gerhard Ritter',
      'url': 'https://en.wikipedia.org/wiki/Gerhard_Ritter'},
     {'birth_year': u'1963',
      'gender': 'Male',
      'name': 'Andrew Roberts',
      'url': 'https://en.wikipedia.org/wiki/Andrew_Roberts_(historian)'},
     {'birth_year': u'1928',
      'gender': 'Male',
      'name': 'J. M. Roberts',
      'url': 'https://en.wikipedia.org/wiki/J._M._Roberts'},
     {'birth_year': u'1949',
      'gender': 'Male',
      'name': 'N.A.M. Rodger',
      'url': 'https://en.wikipedia.org/wiki/N.A.M._Rodger'},
     {'birth_year': u'1942',
      'gender': 'Male',
      'name': 'Walter Rodney',
      'url': 'https://en.wikipedia.org/wiki/Walter_Rodney'},
     {'birth_year': u'1860',
      'gender': 'Male',
      'name': 'William Ledyard Rodgers',
      'url': 'https://en.wikipedia.org/wiki/William_Ledyard_Rodgers'},
     {'birth_year': u'1911',
      'gender': 'Male',
      'name': 'Theodore Ropp',
      'url': 'https://en.wikipedia.org/wiki/Theodore_Ropp'},
     {'birth_year': '1945',
      'gender': 'Male',
      'name': 'W.J. Rorabaugh',
      'url': 'https://en.wikipedia.org/wiki/W.J._Rorabaugh'},
     {'birth_year': u'1946',
      'gender': 'Male',
      'name': 'Ron Rosenbaum',
      'url': 'https://en.wikipedia.org/wiki/Ron_Rosenbaum'},
     {'birth_year': u'1936',
      'gender': 'Male',
      'name': 'Charles E. Rosenberg',
      'url': 'https://en.wikipedia.org/wiki/Charles_E._Rosenberg'},
     {'birth_year': u'1903',
      'gender': 'Male',
      'name': 'Stephen Roskill',
      'url': 'https://en.wikipedia.org/wiki/Stephen_Roskill'},
     {'birth_year': '1943',
      'gender': 'Male',
      'name': 'Maarten van Rossem',
      'url': 'https://en.wikipedia.org/wiki/Maarten_van_Rossem'},
     {'birth_year': u'1915',
      'gender': 'Female',
      'name': u'Mar\xeda Rostworowski',
      'url': 'https://en.wikipedia.org/wiki/Mar%C3%ADa_Rostworowski'},
     {'birth_year': u'1858',
      'gender': 'Male',
      'name': 'Theodore Roosevelt',
      'url': 'https://en.wikipedia.org/wiki/Theodore_Roosevelt'},
     {'birth_year': u'1870',
      'gender': 'Male',
      'name': 'Michael Rostovtzeff',
      'url': 'https://en.wikipedia.org/wiki/Michael_Rostovtzeff'},
     {'birth_year': u'1891',
      'gender': 'Male',
      'name': 'Hans Rothfels',
      'url': 'https://en.wikipedia.org/wiki/Hans_Rothfels'},
     {'birth_year': u'1943',
      'gender': 'Female',
      'name': 'Sheila Rowbotham',
      'url': 'https://en.wikipedia.org/wiki/Sheila_Rowbotham'},
     {'birth_year': u'1916',
      'gender': 'Male',
      'name': 'Herbert H. Rowen',
      'url': 'https://en.wikipedia.org/wiki/Herbert_H._Rowen'},
     {'birth_year': u'1903',
      'gender': 'Male',
      'name': 'A. L. Rowse',
      'url': 'https://en.wikipedia.org/wiki/A._L._Rowse'},
     {'birth_year': '1956',
      'gender': 'Female',
      'name': 'Miri Rubin',
      'url': 'https://en.wikipedia.org/wiki/Miri_Rubin'},
     {'birth_year': u'1910',
      'gender': 'Male',
      'name': u'George Rud\xe9',
      'url': 'https://en.wikipedia.org/wiki/George_Rud%C3%A9'},
     {'birth_year': u'1932',
      'gender': 'Male',
      'name': 'R. J. Rummel',
      'url': 'https://en.wikipedia.org/wiki/R._J._Rummel'},
     {'birth_year': u'1903',
      'gender': 'Male',
      'name': 'Steven Runciman',
      'url': 'https://en.wikipedia.org/wiki/Steven_Runciman'},
     {'birth_year': '1950',
      'gender': 'Female',
      'name': 'Leila J.Rupp',
      'url': 'https://en.wikipedia.org/wiki/Leila_J.Rupp'},
     {'birth_year': u'1937',
      'gender': 'Male',
      'name': 'Conrad Russell',
      'url': 'https://en.wikipedia.org/wiki/Conrad_Russell'},
     {'birth_year': u'1920',
      'gender': 'Male',
      'name': 'Cornelius Ryan',
      'url': 'https://en.wikipedia.org/wiki/Cornelius_Ryan'},
     {'birth_year': u'1908',
      'gender': 'Male',
      'name': 'Boris Rybakov',
      'url': 'https://en.wikipedia.org/wiki/Boris_Rybakov'},
     {'birth_year': u'1910',
      'gender': 'Male',
      'name': 'Edgar V. Saks',
      'url': 'https://en.wikipedia.org/wiki/Edgar_V._Saks'},
     {'birth_year': u'1884',
      'gender': 'Male',
      'name': 'Richard G. Salomon',
      'url': 'https://en.wikipedia.org/wiki/Richard_G._Salomon'},
     {'birth_year': u'1904',
      'gender': 'Male',
      'name': 'S. Srikanta Sastri',
      'url': 'https://en.wikipedia.org/wiki/S._Srikanta_Sastri'},
     {'birth_year': u'1879',
      'gender': '',
      'name': 'J. Salwyn Schapiro',
      'url': 'https://en.wikipedia.org/wiki/J._Salwyn_Schapiro'},
     {'birth_year': u'1974',
      'gender': 'Male',
      'name': 'Dominic Sandbrook',
      'url': 'https://en.wikipedia.org/wiki/Dominic_Sandbrook'},
     {'birth_year': None,
      'gender': '',
      'name': 'Usha Sanyal',
      'url': 'https://en.wikipedia.org/wiki/Usha_Sanyal'},
     {'birth_year': u'1945',
      'gender': 'Male',
      'name': 'Simon Schama',
      'url': 'https://en.wikipedia.org/wiki/Simon_Schama'},
     ...]




```python
import pandas as pd
```


```python
historians_data = pd.DataFrame(historians)
```


```python
historians_data.to_csv("wikipedia_historians.csv",encoding = 'utf8')
```

Now we're all done! 
