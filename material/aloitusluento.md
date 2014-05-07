# Aloitustilaisuden esimerkkisovellus

Seuraavassa kiireessä kirjoitettu, oikolukematon tarina aloitusluennolla demonstroidun sovelluksen rakentumisesta. Pahoittelen tekstin huonoutta. Pull requesteilla voi korjailla typoja ja parantaa kieliasua!

Mukana on myös edityneimpiä asioita (kuten direktiivit ja interceptorit) joihin aloitusluennolla ei päästä.

* Sovelluksen Angular-frontendin koodi [täällä](https://github.com/mluukkai/ng-project-frontend) 
* ja Rails-backendin koodi [täällä](https://github.com/mluukkai/ng-project-backend)
* Itse sovellus [http://ng-project-fe.herokuapp.com](http://ng-project-fe.herokuapp.com)

Saattaa olla kannattavaa käydä ennen allaolevaa läpi virallinen [Angular tutoriaali](https://docs.angularjs.org/tutorial), ainakin luvut 1-7.

Järkevin tapa lukea tätä ohjetta lienee rakentaa sovellus pala palalta samalla itse. Ohje olettaa riittävän javascript-tuntemuksen eikä ole kovin perinpohjainen Angularinkaan suhteen.

## sovellus alkuun

Järkevin tapa Angular-projektien tekoon tällä hetkellä lienee projektin hallinta [Yeomanilla](http://yeoman.io/).

Katso ensin aiheeseen liittyvä [video](https://www.youtube.com/watch?v=iUQ1fvdO9GY#t=719)

Ensin asennetaan Yeoman `npm install -g yo` ([laitoksen koneilla vaatii säätöä](https://github.com/tuhoojabotti/NodeJS-ohjelmointiprojekti-k2014/blob/master/ohjeet/aloittaminen.md#yeoman)), Angular-generaattori `npm install generator-angular` ja Grunt `npm install -g grunt-cli`

Tehdään ensin sovellukselle hakemisto, mennään hakemistoon ja luodaan sovellusrunko komennolla <code>yo angular</code>

Käynnistetään komennolla <code>grunt serve</code>, ja mennään sovelluksen sivulle [http://127.0.0.1:9000](http://127.0.0.1:9000)

Ruvetaan sitten tutustumaan sovelluksen rakenteeseen.

Komennon <code>yo angular</code> suorittamisen myötä syntyy projektirunko:

<pre>
bower.json        - riipuvuuksien määrittely  
Gruntfile.js      - käännös ym taskien konfigurointi
app               
  index.html      - sivun layout 
  scripts         - javascript
    controllers   - kontrollerit
      main.js     
    app.js        - pää
  views           - alisivujen näkymät
    main.html
  styles          - css
test              - testit
</pre>

_index.html_ on sekavahkolta vaikuttava tiedosto, jonka tarkoituksena on määritellä sovelluksen "layout" ja ladata sovelluksen tarvitsemat js-kirjastot ja css:t.

Ellei kyseessä ole hyvin yksinkertainen sovellus, tiedostoon index.html **ei** yleensä ole tarkoitus laittaa muuta sovelluksen sisältöä koskevaa kuin kaikille sivuille yhteiset asiat, esim. navigaatiopalkki. 

Tiedosto sisältää 2 merkillepantavaa asiaa:
```html
<body ng-app="frontendApp">

    <!-- Add your site or application content here -->

    <div class="container" ng-include="'views/main.html'" ng-controller="MainCtrl"></div>

    ...
</body>
```

body-tagin attribuutti <code>ng-app="frontendApp"</code> määrittelee, että kyseessä on AngularJS-sovellus, jonka toiminnan määrittelee moduuli _frontendApp_.

div-tagiin lisätyt attribuutit määrittelevät että ko. kohtaan renderöidään näkymätemplate _main.html_ ja siihen liittyy *kontrolleri* nimeltään <code>MainCtrl</code>.

Hakemistossa _app/views_ sijaitsevassa näkymätemplatessa
*main.html* on kaikenlaista sisältöä. Tehdään siten, että kopioidaan mainissa oleva navigaatiopalkki tiedostoon index.html ja luovutaan mainin "includaamisesta", index.html tulee näin muotoon:

```html
  <body ng-app="frontendApp">
    <!--[if lt IE 7]>
      <p class="browsehappy">You are using an <strong>outdated</strong> browser. Please <a href="http://browsehappy.com/">upgrade your browser</a> to improve your experience.</p>
    <![endif]-->

    <!-- Add your site or application content here -->

    <div class="container" ng-controller="MainCtrl">
      <div class="header">
        <ul class="nav nav-pills pull-right">
          <li class="active"><a ng-href="#">Home</a></li>
          <li><a ng-href="#">About</a></li>
          <li><a ng-href="#">Contact</a></li>
          </ul>
        <h3 class="text-muted">frontend</h3>
      </div>

      <h2>List here some stuff</h2>
    </div>
        
    ...
</body>
```

Teemme siis sovelluksemme hieman hyvien tapojen vastaisesti tiedostoon _index.html_.

Angularissa näkymätemplateihin (eli html-tiedostoihin, kuten index.html) voidaan kirjoittaa <code>{{}}</code>-merkkien sisään Angular-koodia. Ennenkuin sivu näytetään, koodi evaluoidaan ja renderöidään html:n sekaan. 

Kokeillaan. Muutetaan index.html:ää seuraavasti:

```html
	<h2>List here some stuff</h2>

      {{1+2+3}}

      {{"pekka".length}}

      {{awesomeThings}}
```

<code>awesomeThings</code> on taulukko, jonka kontrolleri <code>MainCtrl</code> on liittänyt sivuntemplaten *scopeen*.

Kontrollerin koodi on seuraava:

```javascript
angular.module('frontendApp')
  .controller('MainCtrl', function ($scope) {
    $scope.awesomeThings = [
      'HTML5 Boilerplate',
      'AngularJS',
      'Karma'
    ];
  });
```

Kontrolleri liitetään tiedostossa *app.js*-määriteltyyn moduuliin. app.js on yksinkertainen:

```javascript
angular.module('frontendApp', []);
```

Huomaa että moduulin määrittelyssä on mukana [] kun taas kontrollerissa moduliin viitataan ilman []:ää.

Kontrolleri oltaisiin voitu kirjoittaa myös seuraavasti:

```javascript
var app = angular.module('frontendApp')
  
app.controller('MainCtrl', function ($scope) {
    $scope.awesomeThings = [
      'HTML5 Boilerplate',
      'AngularJS',
      'Karma'
    ];
  });
```

eli ensin talletetaan muuttujaan <code>app</code> 
viittaus sovelluksen määrittelemään moduuliin. Sen jälkeen moduuliin _liitetään_ kontrolleri nimeltään 'MainCtrl'. 

Kontrolleri määritellään anonyymin funktion avulla.
Funktion parametrina on <code>$scope</code>. Parametrin arvona tulee olemaan kontrollerin hallinnoiman sivun osan "scope", eli Angular-magiaan kuuluva mystinen liima, joka "liittää" näkymätemplatet (eli html-tiedostot) ja kontrollerit toisiinsa. 

[Scope](https://docs.angularjs.org/guide/scope) on aika syvällinen asia. Sovelluksella on yleensä useita scopeja, scopet voivat olla järjestäytyneet hierarkkisesti, eli scopella voi olla lapsiscopeja ym... Jätämme scopejen tarkemman käsittelyn harjoitustehtäväksi.

Angular *injektoi* scopen automaattisesti kontrollerille, eli kun metodia kutsutaan, parametrilla on maagisesti oikea arvo. Parametrin nimi on merkityksellinen, jos parametri olisi nimetty eri tavalla, ei sille injektoituisi oikeaa arvoa (eli kontrollerin hallinnoiman sovelluksen osan scopea). Angular siis antaa parametrille arvoksi nimenomaan scopen sen nimen <code>$scope</code> perusteella.

Palataan näkymätemplateen. Muutetaan sen sisältöä seuraavasti:

```html
      <h2>List here some stuff</h2>

      <ul>
        <li ng-repeat="thing in awesomeThings">
          {{thing}}     
        </li>
      </ul>
```

Käytössä on ehkä Angularin eniten käytetty *direktiivi* <code>ng-repeat</code>. Kuten arvata saattaa, direktiivi saa aikaan sen, että li-elementti monistuu siten, että jokaiselle läpikäytävän kokoelman alkiolle muodostuu elementistä oma kopio.

## Kommunikointi backendin kanssa

Sen sijaan että näyttäisimme sivulla listan kontrollerissa taulukkoon kovakoodattuja merkkijonoja, haluamme näyttää sovelluksessa internetistä, esim. sovelluksemme backendistä ladattavia asioita.

Törmäämme tässä kohdassa selainten rajoitteisiin: selain ei voi ladata vapaasti resursseja muista domainista kuin siitä, miltä html-sivu on ladattu, ks. [http://en.wikipedia.org/wiki/Same_origin_policy](http://en.wikipedia.org/wiki/Same_origin_policy). Usein kuitenkin tilanne on se, että haluamme käyttää jotain ulkopuoleisia rajapintoja tai oma backendimme joka ei sijaitse samassa domainissa kuin mistä frontend-koodi haetaan.

Ratkaisuja ongelmaan on muutamia, käytämme näistä järkevintä eli
* [CORSia (Cross-Origin resource sharing)](http://en.wikipedia.org/wiki/Cross-Origin_Resource_Sharing)

Käytännössä CORS toteutetaan HTTP-headereiden avulla.

CORS edellyttää aina sitä, että sovelluksen käyttämä backend tai palvelu on konfiguroitu sopivalla tavalla. Aina ei näin ole ja joudutaan käyttämään muita ratkaisuja esim. [JSONP:tä](http://en.wikipedia.org/wiki/JSONP). 

Käytössämme oleva [Rails-backend](https://github.com/mluukkai/ng-project-backend) on konfiguroitu sallimaan CORS. Konfiguraatio on hyvin helppoa gemin [rack-cors](https://github.com/cyu/rack-cors) avulla.

Myös frontendin puolella tarvitaan pieni temppu.

Muutetaan sovelus-moduulin määrittelevä *app.js* seuraavaan muotoon:

```javascript
var app = angular.module('frontendApp', []);

app.config(function($httpProvider) {
    $httpProvider.defaults.useXDomain = true;
    delete $httpProvider.defaults.headers.common['X-Requested-With'];
});
```

Koodirivit muuttavat hieman sitä, miten Angularin tarjoama <code>$http</code> palvelu toimii oletusarvoisesti.

Muutetaan nyt sovelluksen kontrolleria (tiedostossa main.js) seuraavasti:

```javascript
var app = angular.module('frontendApp')
  
app.controller('MainCtrl', function ($scope, $http) {
    $scope.awesomeThings = [
      'HTML5 Boilerplate',
      'AngularJS',
      'Karma'
    ];

    $http.get('http://ng-project-backend.herokuapp.com/api/blogs.json').success( function(data, status, headers, config) {
    	console.log(data)
    	$scope.entries = data;
    });
});
```

Kontrollerille *injektoidaan* nyt scopen lisäksi myös <code>$http</code>, joka on Angularin valmiiksi tarjoama HTTP:n käytön abstrahoiva palvelu, ks [https://docs.angularjs.org/api/ng/service/$http](https://docs.angularjs.org/api/ng/service/$http). 

Palvelun käyttö on helppoa. Kutsumme sen metodia *get* parametrina kohdeosoite, ja määrittelemme callback-fuktion,
jota kutsutaan operaation onnistuessa. 

Olisimme voineet määritellä callbackin myös epäonnistuneeseen tapaukseen:

```javascript
    $http.get('http://osoite').success( function(data, status, headers, config) {

    }).error( function(data, status, headers, config) {

    });
```

Callback siis liittää palvelimen lähettämän datan scopeen asetettuun muuttujaan <code>entries</code>. Muutetaan näkymää siten, että se näyttää palvelimen palauttaman datan:

```html
	<h2>Blog entries</h2>

	<div>
	<div ng-repeat="entry in entries">
	  <h4>{{entry.subject}} by {{entry.user}} </h4>
	      
	  <blockquote>
	    {{entry.body}}
	  </blockquote>    
	</div>
	</div>
```

Palvelin palauttaa siis taulukon json-muotoisia olioita, joilla on mm. kentät subject, user ja body.

## lomakkeet

Tehdään sovellukseemme mahdollisuus blogien kirjoittamiseen. Luodaan ensin näkymään formi:

```html
      <h2>Create a new entry</h2>

      <form ng-submit="createBlog()">
        name: <input type="text" ng-model="blog.user"></input>        
        <br>
        subject: <input type="text" ng-model="blog.subject"></input>
        <br>
        body:
        <input type="text" ng-model="blog.body"></input>
        <br>
        <button>create</button>
      </form>

      {{blog}}
```

Lomake ei vielä tee mitään ja on ulkoasultaan ruma. Tarkastellaan kuitenkin paria asiaa. Jokaiseen kenttään on liitetty attribuuttidirektiivin *ng-model* avulla arvo, joka vastaa scopessa olevaa muuttujaa (tai sen kenttää) johon kentän arvo tallettuu. 

Lomakkeen alle on lisätty rivi <code>{{blog}}</code>, joka näyttää olion *blog* jonka kenttiin syötteen arvot sidotaan. Jos kokeilet kirjottaa jotain syötekenttiin, pääset todistamaan Angularin two way binding -magiaa.

Nappiin on kiinnitetty *ng-submit*-direktiivin avulla *tapahtumankuuntelija* nimeltään <code>createBlog</code>. Kuuntelijametodia ei ole vielä olemassa, se täytyy määritellä kontrollerissa ja liittää scopeen. Tehdään näin:

```javascript
app.controller('MainCtrl', function ($scope, $http) {
    $http.get('http://ng-project-backend.herokuapp.com/api/blogs.json').success( function(data, status, headers, config) {
    	console.log(data)
    	$scope.entries = data;
    });

    $scope.createBlog = function() {
    	$scope.blog = {}
    	console.log("button clicked")
    }
});
```

Nyt käsittelijä ainoastaan tyhjentää syötekentät ja kirjoittaa konsoliin viesti.

Muutetaan callbackiä siten, että se luo blogi-entryn palvelimelle HTTP POST -kutsulla:

```javascript
    $scope.createBlog = function() {
    	$http.post('http://ng-project-backend.herokuapp.com/api/blogs.json', $scope.blog).success( function(data, status, headers, config) {
    		$scope.entries.push(data)
    	});
    	$scope.blog = {}
    }
```

Kutsu siis tehdään <code>$http</code>-palvelun avulla antamalla kohdeurlin lisäksi lähetettävä dataolio parametriksi. Onnistuneen tapauksen takaisinkutsussa laitetaan luotu blogikirjoitus blogien listalle, jotta myös sivun käyttäjä näkee uuden blogin.

## omat palvelut

Kontrollerin koodi on nyt sikäli ikävää, että siihen on kovakoodattu backendin osoite. Kontrolleri myös käyttää suoraan $http-palvelua ja tämä hankaloittaa kontrollerin testaamista. 

Määritellään oma palvelu, joka piilottaa kontrollerilta nämä alhaisen tason detaljit.

Ideana on määritellä palvelu <code>Blogs</code>, jota kontrolleri voi käyttää seuraavaan tyyliin: 

```javascript
    Blogs.all().success( function(data, status, headers, config) {
    	console.log(data)
    	$scope.entries = data;
    });

    $scope.createBlog = function() {
    	Blogs.create($scope.blog).success( function(data, status, headers, config) {
    		$scope.entries.push(data)
    	});
    	$scope.blog = {}
    }
```

Palvelut ovat käytännössä singleton-olioita, joita on mahdollista *injektoida* esim. kontrollereihin.

Palvelujen määrittelytapoja on muutamia, käytämme seuraavassa tehdasmetodia:

```javascript
app.factory('Blogs', function($http){
    var URL = 'https://ng-project-backend.herokuapp.com/api/blogs.json'; 

    var blogsService = {};
    
    blogsService.all = function(){
    	return $http.get(URL)	
    } 

    blogsService.create = function(data){
    	console.log("called")
    	return $http.post(URL, data)	
    } 

    return blogsService;
 
});
```

Palvelu siis rekisteröidään sovellusmoduulille käyttämällä metodia *factory*. Parametrikseen metodi saa pavelun nimen ('Blocks') lisäksi metodin joka täytyy palauttaa palvelun määrittelevä olio. Olio määritellään 'normaaliin' javascript-tyyliin. Koska palvelu käyttää *$http*-palvelua, on se injektoitava metodin *factory* parametrina olevan funktion parametrina.

Jotta määritelty palvelu saadaan kontrollerissa käyttöön, on se injektoitava kontrollerille.
 Kontrollerilla ei ole enää riippuvuutta $http-palveluun, joten se voidaan poistaa injektoitavien listalta:

```javascript
app.controller('MainCtrl', function ($scope, Blogs) {

    Blogs.all().success( function(data, status, headers, config) {
    	$scope.entries = data;
    }); 

    $scope.createBlog = function() {
    	Blogs.create($scope.blog).success(function(data, status, headers, config) {
    		$scope.entries.push(data);
    	});
    	$scope.blog = {}
    }
});
```

Koska käyttämämme Rails-resurssi noudattaa RESTFull-konventioita, olisimme myös voineet käyttää $http-palvelun sijaan Angularin valmista
[$resource](https://docs.angularjs.org/api/ngResource/service/$resource)-palvelua, jolloin olisimme selvinneet hivenen helpommalla. Tosin sovelluksessamme $resource-palvelusta ei ole kovin suurta hyötyä sillä aksessoimme _blogs_-resurssia vain kahda metodi/url-kombinaatiota käyttäen.

Kirjoitimme esimerkissä palvelun määrittelevän koodin samaan tiedostoon missä kontrollerin määritelmä sijaitsee. Jos sovellus alkaa kasvaa, kannattaa kontrollerit ja palvelut määritellä omissa tiedostoissaan.

## hienosäätöä

Lomakkeemme on nyt koko ajan näkyvillä. Haluisimme sen näkyville vain tarvittaessa, esim. jotain painiketta klikkaamalla.

Muutetaan näkymää seuraavasti

```html
      <h2 ng-click="formVisible = !formVisible">Create a new entry</h2>

      <form ng-show="formVisible">

        <!-- täällä samat kuin ennen -->

      </form>
```

Muutetaan vielä kontrolleria lisäämällä sinne pari riviä:

```javascript
app.controller('MainCtrl', function ($scope, Blogs) {
    // lomake ei aluksi näkyvissä
	  $scope.formVisible = false;

    Blogs.all().success( function(data, status, headers, config) {
    	$scope.entries = data;
    }); 

    $scope.createBlog = function() {
    	Blogs.create($scope.blog).success(function(data, status, headers, config) {
    		$scope.entries.push(data);
    	});
    	// piilota kun uusi blogi-entry luotu
    	$scope.formVisible = false;
    	$scope.blog = {}
    }

});
```

Liitimme nyt lomakkeeseen direktiivin *ng-show*, jonka avulla voidaan määritellä milloin sen sisältävä elementti on näkyvissä. Alustimme lomakkeen näkyvyyttä kontrolloivan muuttujan <code>formVisible</code>arvoksi *false*. Otsikkoon "Create new entry" liitettiin klikkauksenkuuntelija direktiivillä *ng-click*:

```html
      <h2 ng-click="formVisible = !formVisible">Create a new entry</h2>
```

klikatessa tekstiä suoritetaan aina <code>formVisible = !formVisible</code> koodi joka kääntää näkyvyyttä säätelevän muuttujan arvon toisinpäin. 

Olisimme voineet määritellä klikkauksen kuuntelijan myös metodina

```html
      <h2 ng-click="toggleVisibility()">Create a new entry</h2>
```

jos olisimme liittäneet metodin kontrollerissa scopeen:

```javascript
app.controller('MainCtrl', function ($scope, Blogs) {
    $scope.formVisible = false;

    $scope.toggleVisibility = function(){
      $scope.formVisible = !$scope.formVisible;
    }
    // ...
)}
 ```

Jätämme kuitenkin alkuperäisen klikkauksenkuuntelijan käyttöön. 

Hyödynnetään [bootstrapia](http://getbootstrap.com/) ja tehdään lomakkeesta hieman siistimmän näköinen (valitettavasti HTML:stä tulee samalla aikamoista sotkua):

```html
      <a class="btn btn-primary btn-lg btn-block" ng-click="formVisible = !formVisible" ng-hide="formVisible">Create a new entry</a>

      <form ng-show="formVisible" role="form">
        <div class="form-group">
          <input class="form-control" type="text" ng-model="blog.user" placeholder="your name"></input>  
        </div>    
        <div class="form-group">
          <input class="form-control" type="text" ng-model="blog.subject" placeholder="subject"></input>  
        </div>   
        <div class="form-group">  
          <textarea class="form-control" rows="3" ng-model="blog.body" placeholder="content"></textarea>
        </div>
        <button class="btn btn-primary" ng-click="createBlog()">create</button>
        <button class="btn btn-default" ng-click="formVisible=!formVisible">cancel</button>

      </form>
```

Huomaa, että nyt käytössä *ng-shown* 'vastakohtadirektiivi' <code>ng-hide</code> ,jonka avulla elementin voi piilottaa silloin kuin tietty ehto evaluoituu todeksi.

## deployment

Nyt on korkea aika deployata sovellus herokuun. Toimenpide on helppo, ensimmäisen vaiheen ohje löytyy [täältä](http://www.lars-schenk.com/deploying-a-yeoman-angular-app-to-heroku/1661).

Toinen vaihe on "kääntää" sovellus komennolla <code>grunt build</code>. Tällöin sovelluksen css:ien ja js-koodin osalta minimoitu versio syntyy hakemistoon _dist_. Oletusarvoisesti _.gitigonre_ tiedosto ignoroi ko. hakemiston, ignorointi tulee poistaa!

Seuraavaksi luodaan tavanomaiseen tapaan heroku-sovellus (komennolla <code>heroku create</code>) ja pushataan commitoitu repositorio (jossa siis on oltava mukana myös _dist_-hakemisto) herokuun. 

Sovelluksen pitäisi sitten toimia: [http://ng-project-fe.herokuapp.com](http://ng-project-fe.herokuapp.com)

## filtteröinti

Parannellaan hieman blogien listaa. Angularissa on runsaasti valmiiksi määriteltyjä [filttereitä](https://docs.angularjs.org/api/ng/filter). Osa filttereistä on tarkoitettu taulukoiden käsittelyyn. Aloitetaan järjestämällä blogientryt siten, että uusimmat tulevat ylimmäksi. Tämä onnistuu helposti [orderBy:n](https://docs.angularjs.org/api/ng/filter/orderBy) avulla:

```html
      <div>
        <div ng-repeat="entry in entries | orderBy:'id':true">
          <h4>{{entry.subject}} by {{entry.user}} </h4>

          <blockquote>
            {{entry.body}}
          </blockquote>    
        </div>
      </div>
``` 

Backend antaa jokaiselle entrylle uniikin id:n. Id kasvaa inkrementaalisesti, joten järjestämme entryt id:n perusteella käänteisesti: <code>entries | orderBy:'id':true</code>. Huom: olioiden id:n perusteella tapahtuva järjestäminen ei ole välttämättä järkevää, parempi olisi liittää jsoniin mukaan olioiden luontiaika.

Tehdään vielä sovellukseen mahdollisuus rajoittaa näytettäviä blogientryjä tekstihaun perusteella. Käytetään filtteriä nimeltä [filter](https://docs.angularjs.org/api/ng/filter/filter):

```html
     <input placeholder="write somthing to filter entries" class="form-control" ng-model="criteria"></input>

      <div>
        <div ng-repeat="entry in entries | filter:criteria | orderBy:'id':true">
          <h4>{{entry.subject}} by {{entry.user}} </h4>

          <blockquote>
            {{entry.body}}
          </blockquote>    
        </div>
      </div>
```

Tekstikenttään kirjoitettu teksti tallettuu nyt scopen muuttujaan _criteria_. Filtteri käyttää tätä 
rajoittamaan entryjä: <code>entry in entries | filter:criteria | orderBy:'id':true</code>. Filtteröinnin jälkeen entryille suoritetaan vielä järjestäminen. Eli kuten näemme, filtterejä voidaan helposti ketjuttaa.

Filtterejä on myös helpohko kirjoittaa itse.

## routing

Olemme kirjottaneet koko sovelluksen yhden kontrollerin alaisuuteen, samaan näkymätemplateen. Yleensä näin ei kannata tehdä. Jos esim. tekisimme yksittäiselle blogille oman sivun (joka mahdollistaisi blogin editoinnin jne), kannattaisi tälle toiminnolle muodostaa oma näkymätemplate ja kontrolleri. Järkevä tapa hoitaa asia on Angularin *reititysmekanismin* käyttö. Angularin [tutoriaali](https://docs.angularjs.org/tutorial/step_07) esittelee aihetta ansiokkaasti.

## direktiivit 

Angularin magia saadaan aikaan toisaalta kontorollerien ja näkymätemplaten jakaman scopen avulla, toisaalla taas _direktiiveillä_,joita kirjoittamalla määritellään miten näkymätemplatejen tulee toimia. Angularissa on runsaasti valmiita direktiivejä, mm. jo meille tutut _ng-repeat_, _ng-model_, _ng-click_, _ng-show_ jne...

Angularin valmiilla direktiiveillä päästään jo jonnekin asti, mutta sovelluskehyksen todellinen voima tulee siitä, että se mahdollistaa omien, lähes mielivaltaisella tavalla toimivien direktiivien määrittelemisen. Direktiivit sopivat moneen tarkoitukseen, niiden avulla on mm. mahdollista määritellä uusia HTML-elementtejä.

Monet valmiit direktiivit, esim. _ng-show_ manipuloivat DOM:ia. Direktiivit ovatkin se paikka missä Angular-sovellusten kaiken DOM-manipuloinnin tulee tapahtua. On todella paha Angular-antipatterni koskea millään muotoa DOMiin kontrollereissa.

Direktiivit ovat erittäin syvällinen aihe, katsotaan kuitenkin kohta yhtä esimerkkiä. Toteutetaan kuitenkin ensin sovellukseen sen käyttöä sujuvoittava pieni ominaisuus.

## flash

Sovellus käyttäytyy nyt hieman ikävästi uuden blogipostauksen submittauksen jälkeen. Blogi ainoastaan lisätään ao. listalle ja formi katoaa. Lisätään sovelluksen uuden blogientryn luomisesta kertova "flash"-viesti. 

Lisätään html-templaten ylosaan seuraava:

```html
      <div ng-show="flash" class="alert alert-success">
        {{flash}}
        <span style="float:right" ng-click="flash=null" class="glyphicon glyphicon-remove-sign"></span>
      </div>
```

Luotu elementti siis näyttää scopen muuttujaan _flash_ sijoitetun tekstin _jos_ muuttujassa on jotain ja sen arvo ei ole false (eli sen arvo ei ole falsy).

Nyt blogin luova callback-metodi voi asettaa flashille arvon:

```javascript
    $scope.createBlog = function() {
      Blogs.create($scope.blog).success(function(data, status, headers, config) {
        $scope.entries.push(data);
      });
      // asetetaan muuttujalle flash arvo
      $scope.flash = "A new blogentry '"+$scope.blog.subject+"'' created"

      $scope.formVisible = false;
      $scope.blog = {}
    }
```

Flash-viestiin liittyvän rastin klikkaaminen saa viestin pois ruudulta rastin yhteyteen määritellyn muutujan *flash* nollaava klikkauksenkäsittelijän <code>ng-click="flash=null"</code> ansiosta. 

## omat direktiivit 

Ratkaisu toimii, mutta html-template alkaa näyttää koko ajan ikävämmältä.

Eristetän flash-viestin näyttäminen _direktiiviksi_. Luodaan oma html-elementin tapaan toimiva direktiivi, jota voi käyttää aluksi seuraavasti

```html
   <flash></flash>
```

Direktiivi määritellään seuraasti:

```javascript
app.directive('flash', function() {
  return {
      restrict: 'AE',
      templateUrl: 'views/flash.html'
  };
});
```

Kuten arvata saattaa, direktiiviin liittyvä html on määritelty tiedostossa _views/flash.html_. Tiedoston sisältö on sama kuin aiemmin suoraan templateen index.html kirjoittamamme, eli:

```html
<div ng-show="flash" class="alert alert-success">
  {{flash}}
  <span style="float:right" ng-click="flash=null" class="glyphicon glyphicon-remove-sign"></span>
</div>
```

Direktiivin määrittelyssä oleva <code>restrict: 'AE'</code> tarkoittaa, että direktiiviämme voi käyttää joko suoraan html-elementin tapaan:


```html
   <flash></flash>
```

tai esim. esim div-elementin attribuuttina:

```html
   <div flash></div>
```

Määrittelemämme direktiivi toimii samassa scopessa kuin muu sivu, tämän takia muuttuja _flash_ on suoraan viitattavissa direktiivin sisällä.

Direktiivi on kuitenkin hieman ikävä, sillä se edellyttää 'flashattavan' tekstin olevan muuttujassa _flash_ ja käytetty tyyli on kovakoodattu.

Generalisoidaan ratkaisua hieman. Tehdään ensin minkä tahansa scopessa olevan muuttujan flashaaminen mahdolliseksi.

Flashattava muuttuja määritellään attribuutin _message_ avulla.

```html
  <flash message='flash'></flash> 
```

Flashattavaan tekstin sisältävään muuttujaan viitataan direktiivin sisällä nyt nimellä _message_, eli templatea on muutettava seuraavasti:

```html
<div ng-show="message" class="alert alert-success">
  {{message}}
  <span style="float:right" ng-click="message=null" class="glyphicon glyphicon-remove-sign"></span>
</div>
```

Myös direktiivin määritelevä koodi muuttuu:

```javascript
app.directive('flash', function() {
  return {
      restrict: 'AE',
      scope: {
        message:'='
      },
      templateUrl: 'views/flash.html'
  };
});
```

Muutoksena edelliseen on direktiivin nyt määritelty eksplisiittisesti *oma scope*. Direktiivi ei enää oletusarvoisesti näe mitään sen sijaintipaikan scopen muuttujia tai funktioita. Merkinnän <code>message:'='</code> avulla välitetään *viite* attribuutin message 'arvona' olevaan scopen muuttujaan direktiiville. Eli koska templatessa on

```html
  <flash message='flash'></flash> 
```

viittaa direktiivin sisällä muuttuja _message_ samaan olioon, kuin muualla sovelluksen scopessa muuttuja _flash_, jota kontorolleri käyttää asettamaan flashattavan tekstin:

```javascript
    $scope.createBlog = function() {
        // ...
        $scope.flash = "A new blog entry '"+$scope.blog.subject+"'' created"
```

## domin manipulointi ja direktiivin link-funktio

Tällä hetkellä flashviesti on kovakoodattu käyttämään
bootstrapin tyyliä [alert-success](http://getbootstrap.com/components/#alerts). 

Muutamme direktiiviä siten, että flashviestin tyyli on mukattavissa attribuutin _alert_ avulla:

```html
      <flash message='flash' type='success'>
      </flash>
```
```html
      <flash message='dangerousThing' type='warning'>
      </flash>
```

Muutetaan direktiivin templatea hieman poistamalla siitä flashin tyypin tarkentanut luokka:

```html
<div ng-show="message" class="alert">
  {{message}}
  <span style="float:right" ng-click="message=null" class="glyphicon glyphicon-remove-sign"></span>
</div>
```

Direktiivin määrittelemän funktion laajennus tapahtuu seuraavasti:

```javascript
app.directive('flash', function() {
  return {
      restrict: 'AE',
      replace: true,
      scope: {
        message:'='
      },
      templateUrl: 'views/flash.html',
      link: function(scope, elem, attrs) {
        if ( attrs['type']!=undefined) {
          elem.addClass('alert-'+attrs['type'])
        } else {
          elem.addClass('alert-success')
        } 
      }
  };
});
```

Ensinnäkin lisäsimme määrittelyn <code>replace: true</code>. Tämä oltaisiin voitu tehdä jo aiemmin. Määrittely saa aikaan sen, että direktiivin template korvaa DOM:ssa kokonaan tägin <code>flash</code>. Ilman määrittelyä direktiivin template sijoitetaan direktiivin määrittelemän tägin sisälle.

Mielenkiintoisempi on nyt mukaan otettu funktio _link_. Linkitysfunktio suoritetaan siinä vaiheessa, kun direktiivin template ja siihen liitetty scope renderöidään HTML-templaten sekaan. Linkitysfunktio saa kolme parametria: _scope_ on direktiivin scope, _elem_ on itse direktiivielementti, meidän tapauksessamme templaten määrittelemä div. Elementti on wrapatty _jquery_-olioksi.
Kolmantena parametrina on direktiivin sisältämät attribuutit taulukkomuodossa. 

Määrittelemämme linkitysmetodi ottaa _type_-attribuutin, muodostaa tästä elementille lisättävän (käytännössä sen värin määrittelevän) luokan ja liittää sen elementtiin. Elementtiä käsitellään Angularin sisältämällä 'minijqueryllä' eli [jqLite:llä](https://docs.angularjs.org/api/ng/function/angular.element) 

Eli jos direktiiviä käytetään seuraavasti

```html
      <flash message='dangerousThing' type='warning'>
      </flash>
```

lisää _link_-funktio flashille luokan 'alert-warning'. Jos tyyppiä ei määritellä:

```html
      <flash message='flash'>
      </flash>
```

liittää _link_-funktio flashille oletusarvoisen luokan 'alert-success'.

Direktiivit ovat erittäin syvällinen aihe ja olemme tässä vasta repäisseet pintaa...


## autentikointi

Päätämme toteuttaa sovellukseen ominaisuuden, jonka avulla (sopimattoman sisällön omaavia) blogikirjoituksia on mahdollista poistaa.

Lisätään jokaisen blogientryn yhteyteen roskisnappi:

```html
      <div>
        <div ng-repeat="entry in entries | filter:criteria| orderBy:'id':true">
          <h4>
            {{entry.subject}} by {{entry.user}} 
            <span 
              style="float:right" 
              ng-click="deleteBlog(entry)" 
              class="glyphicon glyphicon-trash">
            </span>
          </h4>

          <blockquote>
            {{entry.body}}
          </blockquote>    
        </div>
      </div>
```

Nappiin on kiinnitetty klikkauksenkuuntelija *deleteBlog*, joka saa parametrikseen poistettavan blogientryolion.

Klikkauksenkuuntelija määritellään ja kiinnitetään scopeen kontrollerissa:

```javascript
app.controller('MainCtrl', function ($scope, Blogs) {
    // ...

    $scope.deleteBlog = function(entry) { 
      Blogs.delete(entry).success(function(){
        var index = $scope.entries.indexOf(entry)
        $scope.entries.splice(index, 1);
      });
    }

    // ...
}
```

Kuuntelija siis kutsuu *Blogs*-palvelun metodia *delete* (jonka lisäämme kohta palvelulle). Takaisinkutsussa tuhottu blogientry poistetaan myös scopessa olevalta listalta.

Palvelun *Blogs* uusi metodi on suoraviivainen. Blogientryn poistaminen onnistuu helposti tekemällä backendille sopiva HTTP DELETE -operaatio:

```javascript
app.factory('Blogs', function($http){
    var URL_BASE = 'http://ng-project-backend.herokuapp.com/api/blogs'; 

    var blogsService = {};
    
    blogsService.all = function(){
      return $http.get(URL_BASE+".json")  
    } 

    blogsService.create = function(data){
      console.log("called")
      return $http.post(URL_BASE+".json", data) 
    } 

    blogsService.delete = function(data){
      return $http.delete(URL_BASE+"/"+data.id+".json", data) 
    } 

    return blogsService;
});
```

Urlien hieman eroavan loppuosan takia refaktoroimme hieman myös vanhoja metodeja.

Enne kuitenkaan halua että kuka vaan voi poistella blogientryjä. Päätämme, että ainoastaan järkestelmään kirjautuneet adminkäyttäjät voivat tehdä näin.

Backend onkin konfiguritu vastaamaan toistoyritykseen HTTP-statuskoodilla 401 (Unauthorized) poisto yritetään tehdä kirjautumattomana. Konsolista käsin voi havaita että näin todellakin käy.

Järjestelmän käyttäjät rekisteröidään suoraan backendiin sivun
[http://ng-project-backend.herokuapp.com/users](http://ng-project-backend.herokuapp.com/users) kautta. 

Autentikointi tapahtuu siten, että käyttäjänimi (username) ja salasana (password) lähetetään HTTP POST:in avulla osoitteeseen *http://ng-project-backend.herokuapp.com/session*

Jos autentikointi onnistuu, palauttaa palvelin *autentikointitokenin*, jos ei, palauttaa palvelin HTTP statuskoodin 422 (Unprocessable Entity).

Token-perustaisen autentikoinnin periaatetta on selitetty esim. [täällä](https://auth0.com/blog/2014/01/07/angularjs-authentication-with-cookies-vs-token/).

Lyhyesti idea on seuraava:
* fronend lähettää käyttäjätunnus/salasana-parin palvelimmelle
* jos pari on validi, palauttaa palvelin AT:n (authorization token)
* frontend tallettaa AT:n ja *liittää* sen mukaan kaikkiin palvelimelle meneviin pyyntöihin
* jos palvelimelle tehdään pyyntö joka on sallittu vain osalle käyttäjistä, palvelin tarkastaa liittyykö pyyntöön validi AT
  * jos ei, vastaa palvelin HTTP statuskoodilla 401 (unauthorized)
* kun käyttäjä loggaa ulos sovelluksesta, pyytää fronend palvelinta mitätöimään AT:n
  * mitätöinti voi tapahtua myös kuluneen ajan perusteella tms

Sovelluksemme backend olettaa, että AT on liitetty pyynnön
*auth-token* headeriin.

Jotta saamme frontendin toimimaan, tarvitsemme:
* kirjautumisen mahdollistavan lomakkeen
* tavan muistaa autentikoitumisen yhteydessä saatava AT

Aloitetaan lomakkeella. Lisätään se aluksi sattumanvaraiseen kohtaan sivua:

```html
      <form ng-hide="loggedIn.status" ng-submit="login()">
        <input ng-model="credentials.user" placeholder="username"/>
        <input ng-model="credentials.password" placeholder="password"/>
        <button>login</button>
      </form>

      <form ng-show="loggedIn.status">
        {{loggedIn.user}} 
        <button ng-click="logout()">logout</button>
      </form>
```

sivulle on lisätty myös poiskirjatumisnappi.

Scopeen on lisätty muuttuja *loggedIn*, joka sisältää totuusarvoisen kentän *status* jonka arvo on true jos käyttäjä on kirjautuneena. Muuttujan kenttä *user* kertoo kirjautuneen käyttäjän nimen.

Sisään- ja uloskirjautumisesta huolehtivat callbackit ovat seuraavanlaiset:

```javascript
app.controller('MainCtrl', function ($scope, Blogs, Auth) {
    $scope.loggedIn = Auth.logged;

    $scope.logout = function() {
      Auth.logout()
    }

    $scope.login = function(){
      Auth.login($scope.credentials)
      .then( 
        function(data) {
          // success
        },function( data ) {
          // failure
        }
      );
      $scope.credentials = {}
    }

    // ...
}
```

Kontrollerille on nyt injektoitu *Auth*-niminen service. Kuten arvata saattaa, kyseinen palvelu kapseloi kirjautumiseen liittyvät toiminnallisuuden metodiensa *logout* ja *login* avulla. Palveluun liittyy myös olio *logged*, joka kertoo kirjautumisstatuksen. Olio liitetään scopeen nimellä *loggedIn*.

Kirjautumispalvelun metodit ovat suoraviivaisia. Metodiin *login* liittyvä *then* on jotain mitä emme ole aiemmin nähneet.

Thenin parametrina on kaksi callback-metodia. Tässä tilanteessa metodit toimivat lähes samalla tavalla kuin aiemmin käyttämämme $http-kutsuun liittyvät *success* ja *error* callbackit. Then liittyy Angularin [promise]
(https://docs.angularjs.org/api/ng/service/$q) rajapintaan.
Katsomme asiaa hieman tarkemmin Auth-palvelun koodin yhteydessä.

Tällä hetkellä emme tee callbackeissa mitään, joten voisimme jättää ne määrittelemättä.

Auth-palvelun koodi on seuraavassa:

```javascript
app.factory('Auth', function($http){
    var URL = 'http://ng-project-backend.herokuapp.com/session'; 
    var service = {};

    service.logged = {}; 

    service.login = function(credentials) {
      return $http.post(URL, credentials).then(
        function (token) {
          service.logged.status = true;
          service.logged.user = credentials.user;
          $http.defaults.headers.common['auth-token'] = token.data;
          return token.data;
        }
      )  
    } 

    service.logout = function(credentials) {
      return $http.delete(URL).then(
        function () {
          service.logged.status = false;
          service.logged.user = null;
          delete $http.defaults.headers.common['auth-token']
          return null;
        }
      )
    } 

    return service;
});
```

Palveluun on nyt liitetty metodien lisäksi olio *logged*, jonka attribuutin *status* arvoksi asetetaan true kun käyttäjä on kirjautuneena. Samalla *logged*:in attribuutin *user* arvoksi asetetaan kirjautuneen käyttäjän nimi. Kontrolleri siis kiinnittää muuttujan scopeen nimellä *loggenIn*.

Metodin *login* palvelun $http kutsuun liittyy nyt tavanomaisen *success* callbackin sijaan jo edellä nähty *then*. Kyseessä on metodi jonka ensimmäisenä parametrina oleva metodi suoritetaan jos http-pyyntö onnistuu. Jos kutsu epäonnistuu, kutsuttaisiin toisena parametrina olevaa metodia jos sellainen on olemassa. 

Metodi *then* mahdollistaa callback-metodien ketjutuksen siten, että asynkroninen koodi saadaan käyttäytymään synkronisen tapaan. Näin voidaan välttää js-ohjelmoinnissa kovin tyypillinen callback-helvetti.

Theniä voitaisiin käyttää esim. seuraavaa tyyliin:

```javascript
$http.get('/beer/1')
.then(
  function( response ){
    $scope.beer = response.data;
    return $http('/breweries/'+beer.brewery_id)
  })
).then(
  function( response ){
    $scope.brewery = response.data;
    return $http('/concerns/'+brewery.concern_id)
  }) 
).then(
  function( response ){
    $scope.concern = response.data;
  }
)
```

Ensin haetaan olut jonka id on 1, tämän jälkeen olueeseen liittyvän panimon tiedot ja lopulta panimoon liittyvän konsernin tiedot. Kyseessä on oikeastaan monitasoinen callback-kutsu joka saadaan thenien aikaan näyttämään 'synkronisesti' eli peräkkäin suoritetuilta hauilta.

Thenin avulla on myös mahdollista suorittaa kätevästi kaksi eri callback-metodia tiettyyn kutsuun liittyen. Näin tapahtuu myös Auth-palvelussa: 


```javascript
    service.login = function(credentials) {
      return $http.post(URL, credentials).then(
        function (token) {
          service.logged.status = true;
          service.logged.user = credentials.user;
          $http.defaults.headers.common['auth-token'] = token.data;
          return token.data;
        }
      )  
    } 
```

Metodin *post* 'ensimmäinen' callback muuttaa kirjautumis-statuksen todeksi, tallettaa kirjautuneen käyttäjän nimen sekä asettaa palvelimen palauttaman tokenin headerin *auth-token* arvoksi:

<pre>
$http.defaults.headers.common['auth-token'] = token.data;
</pre>

Headerin lisääminen saa aikaan sen, että se liitetään _kaikkiiin_ palvelimelle tehtäviin HTTP-kutsuihin.

Lopuksi token palautetaan mahdolliselle metodin käyttäjälle. Eli jos metodia käytettäisiin kontrollerista seuraavasti:

```javascript
    $scope.login = function(){
      Auth.login($scope.credentials)
      .then( 
        function(data) {
          // print token
          console.log(data)
        }
      );
      $scope.credentials = {}
    }
```

saisi kontrollerin rekisteröimä callback parametriksi Auth-palvelun palvelimelta hakeman autentikointitokenin.

Auth-pavelun metodi *login* toimii yllätyksettömästi. Ensin pyydetään palvelinta tuhoamaan HTTP DELETE -kutsuun liittyvä token, poistaan tokenin sisältämä headeri ja muutetaan kirjautumisstatus.

# allaoleva valmistuu ehkä tiistaina, varmuudella keskiviikko-iltaan mennessä

## interceptorit 

Tällä hetkellä sovellus ei tee mitään jos kirjautumattomana yritetään tuhota blogi. Luonnollinen korjaus tähän olisi pilottaa tuhomaisnappi jos käyttäjä ei ole kirjautuneena:

```html
    <div ng-repeat="entry in entries | filter:criteria| orderBy:'id':true">
      <h4>
        {{entry.subject}} by {{entry.user}} 
        <span ng-show="loggedIn.status"
          style="float:right" 
          ng-click="deleteBlog(entry)" 
          class="glyphicon glyphicon-trash">
        </span>
      </h4>

      <blockquote>
        {{entry.body}}
      </blockquote>    
    </div>
  </div>
```

Päätämme (paremman esimerkin puutteessa) kuitenkin olla piilottamatta tuhoamissymbolia. 

Jos kirjautumaton käyttäjä yrittää poistaa blogin, vastaa palvelin HTTP-statuskoodilla 401. Tällöin suoritetaan http-palvelun delete-kutsuun liitetty *error*-callback. Eli voisimme huomauttaa kirjautumatonta käyttäjää, että operaatio vaatii kirjautumisen: 

```javascript
    $scope.deleteBlog = function(entry) { 
      Blogs.delete(entry).success(function(){
        var index = $scope.entries.indexOf(entry)
        $scope.entries.splice(index, 1);
      }).error(function(){
        alert('you should be logged in')
      });
    }
```

Teemmekin kuitenkin tämän tilanteen kannalta suunnilleen samalla tavalla toimivan, mutta hieman erilaisen ratkaisun, ja error-calbackin sijaan käytämme [interceptoria](https://docs.angularjs.org/api/ng/service/$http). Interceptorit ovat funktioit, joita voidaan määritellä ajettavaksi joko juuri ennen HTTP-pyynnön tekemistä tai juuri ennen HTTP-pyynnön vastauksen kutsujalle palauttamista. 

Esim. seuraava interceptori kirjottaa konsoliin jokaisen HTTP-pyynnön palauttaman statuskoodin:

```javascript
app.factory('myHttpInterceptor', function($q) {
    return {
     'response': function(response) {
        console.log(response.status)

        return response;
      },
     'responseError': function(response) {
        console.log(response.status)

        return $q.reject(response);
      }
    };
  });
```

Interceptori on määritelty nyt palveluna nimeltään myHttpInterceptor. Palvelulle on määritelty erikseen metodit jotka suoritetaan onnistuneen ja epäonnistuneen HTTP-kutsun yhteydessä. Onnistuneen kutsun yhteydessä suoritettava metodi *response* palauttaa parametrinaan saamansa response-olion eteenpäin. 
Epäonnistuneen kutsun yhteyteen määritelty metodi *responseError* rejektoi pyynnön palvelun *$q* avulla. 

Interceptori pitää vielä ottaa käyttöön config-metodin avulla:

```javascript
app.config(function($httpProvider) {
  $httpProvider.interceptors.push('myHttpInterceptor')
});
```

Interceptoreita voi tarvittaessa olla useita:

```javascript
app.config(function($httpProvider) {
  $httpProvider.interceptors.push('firstInterceptor');
  $httpProvider.interceptors.push('secondInterceptor');
  $httpProvider.interceptors.push('thirdInterceptor');
});
```

Määritellään nyt interceptori, joka kehoittaa alertin avulla käyttäjää kirjautumaan *jos* suorittaa kirjatutumista vaativan toimenpideen (eli statuskoodi on 401 unauthorized). Tässä tapauksessa interceptori palauttaakin response-olion, eli operaatio ei enää kutsujan kannalta näytä epäonnistuneelta. Jos statuskoodi on joku muu, interceptori ei tee mitään eli palauttaa edelleen rejektoituna pysyvän pyynnön.

```javascript
app.factory('loginInterceptor', function($q) {
    return {
     'responseError': function(response) {
        if ( response.status==401 ) {
          alert('you should be logged in')
          return response;
        }

        return $q.reject(response);
      }
    };
  });
```

Interceptorin avulla voisimme, esim. avata eilaillisen HTTP-pyynnön yhteydessä automaattisesti login-dialogin. 

Onnistuneen kirjautumisen yhteydessä olisi interceptorista käsin myös mahdollista suorittaa automaattisesti aiemmin epäonnistunut pyyntö.

Viimeistellään sovellus vielä nopeasti siten että tehdään sovellukseen kirjautumisdialogi jonka interceptori avaa jos kirjaantumaton käyttäjä yrittää tehdä kirjautumista vaativan operaation. Ratkaisu ei ole paristakaan syystä kovin tyylikäs ja ei itseasiassa tarvitsisi edes interceptoria. Ajan puutteen vuoksi parempi ratkaisu (jossa intercepori odottaisi kirjautumisen tuloksen ja uusisi kielletyn operaation) jätetään harjoitustehtäväksi.

Tehdään direktiivinä määritelty modaalidialogi [tätä sivua](http://adamalbrecht.com/2013/12/12/creating-a-simple-modal-dialog-directive-in-angular-js/) mukaillen. 

Huom: esim. bootstrap tarjoaa valmiita ratkaisuja dialogien tekemiseen, ks. [http://angular-ui.github.io/bootstrap/]

Template on seuraava:

```html
<div class='ng-modal' ng-show='modal.visible'>
  <div class='ng-modal-overlay'></div>
  <div class='ng-modal-dialog' ng-style='dialogStyle'>

    <h2>Please log in</h2>

      <form ng-submit="login()" role="form">
      <div class="form-group">
        <input class="form-control" type="text" ng-model="credentials.user" placeholder="username"/>
      </div>  
      <div class="form-group">
        <input class="form-control" ng-model="credentials.password" placeholder="password"/>
      </div>  
        <button class="btn btn-primary" ng-click="createBlog()">login</button>
        <button ng-click='hideModal()' class="btn btn-default">cancel</button>
      </form>

  </div>
</div>
```

Dialogi siis sisältää lomakkeen ja käyttää kahta callbackiä riippuen painetaanko *login* vai *cancel*. Lomake on näkyvissä jos muuttujan *modal* kentän *visible* arvo on tosi.

Direktiivin määrittelevä koodi on seuraava

```javascript
app.directive('modal', function() {
  return {
    restrict: 'E',
    controller: function($scope, Auth){
      $scope.hideModal = function() {
        scope.modal.visible = false;
      };
      $scope.login = function(){
        Auth.login($scope.credentials)
        $scope.modal.visible = false;
      }
    },
    templateUrl: 'views/modal.html',
  };
});
```

Määrittely on suoraviivainen, direktiivin kontrollerin avulla määritellään direktiivin käyttämien klikkaustenkuuntelijoiden toiminnallisuus. Kontrollerille on injektoitu Auth-palvelu palvelimen kanssa käytävää kommunikointia varten.

Interceptori on suoraviivainen, se asettaa dialogin näkyvyyden määrittelevän muuttujan arvoksi true:

```javascript
app.factory('loginInterceptor', function($q, $rootScope) {
    return {
     'responseError': function(response) {
        if (response.status==401) {
          $rootScope.modal.visible = true
        }

        return $q.reject(response);
      }
    };
  });
```

Tässä tulee esille yksi ratkaisumme likaisista puolista, näkyvyyden määrittelevä muuttuja on jouduttu kiinnittämään sovelluksen kaikkien scopejen 'äitiin', eli rootScopeen, sillä interceptorin sisältä ei pystytä näkemään muihin sovelluksen scopeihin. Nyrkkisääntönä on, että rootscopea tulee käyttää erittäin harkiten. 

Kontrolleri asettaa

```javascript
app.controller('MainCtrl', function ($rootScope, $scope, Blogs, Auth) {
  //...
  $rootScope.modal.visible = false;
  //...
}
```

Näkymätemplatea siloitellaan vielä liittämällä siihen modaali ja laittamalla "logout"-nappi navigaatiopalkkiin:

```html
    <div class="container" ng-controller="MainCtrl">
      <div class="header">
        <ul class="nav nav-pills pull-right">
          <li class="active"><a ng-href="#">Home</a></li>
          <li><a ng-href="#">About</a></li>
          <li><a ng-href="#">Contact</a></li>
          <li ng-show="loggedIn.status">
            <a ng-click="logout()" ng-href="#">Logout</a>
          </li>
          </ul>
        <h3 class="text-muted">TKTLng</h3>
      </div>

      <modal/></modal>
```

Esimerkkisovelluksemme on nyt valmis.