## sovelluksen asennus

Tehdään hakemisto

Ks. ensin [https://www.youtube.com/watch?v=iUQ1fvdO9GY#t=719](https://www.youtube.com/watch?v=iUQ1fvdO9GY#t=719)

Käynnistetään komennolla <code>grunt server</code>, ja mennään sovelluksen sivulle [http://127.0.0.1:9000](http://127.0.0.1:9000)

Syntyy aluksi sekavahkolva vaikuttava projektirunko:

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

index.html on sekavahkolta vaikuttava tiedosto, jonka tarkoitus on määritellä sovelluksen "layout" ja ladata sivujen tarvitsemat js-kirjastot ja css:t.

index.html:n **ei** yleensä ole tarkotus tehdä muuta sovelluksen sisältöä koskevaa kuin kaikille sivuille yhteiset asiat, esim. navigaatiopalkki.


Tiedosto sisätää 2 tärkeää asiaa:
```html
<body ng-app="frontendApp">

    <!-- Add your site or application content here -->

    <div class="container" ng-include="'views/main.html'" ng-controller="MainCtrl"></div>

    ...
</body>
```

body-tagin attribuutti <code>ng-app="frontendApp"</code> määrittelee että kyseessä on Angular-sovellus, jonka toiminnan määrittelee moduuli "frontendApp".

div-tagiin lisätyt attribuutit määrittelevät että ko. kohtaan renderöidään näkymätemplate main.html ja siihen liittyy *kontrolleri* nimeltä <code>MainCtrl</code>.

Mainissa on kaikenlaista sisältöä. Tehdään siten, että kopioidaan mainissa oleva navigaatiopalkki index.html:n ja luovutaan mainin "includaamisesta", index tulee muotoon:

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
```

Angularissa näkymätemplateihin (eli html-tiedostoihin, kuten index.html:) voidaan kirjoittaa <code>{{}}</code>-merkkien sisään Angular-koodia, jonka evaluoidaan ja renderöidään html:n sekaan. 

Kokeillaan. Muutetaan index.html:ää:

```html
	<h2>List here some stuff</h2>

      {{1+2+3}}

      {{"pekka".length}}

      {{awesomeThings}}
```

<code>awesomeThings</code> on taulukko, jonka kontrolleri <code>MainCtrl</code> on liittänyt sivuntemplaten *scopeen*

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

Kontrolleri liitetään tiedostossa app.js-määriteltyyn moduuliin. app.js on yksinkertainen:

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
Funktion parametrina on <code>$scope</code>. Parametrin arvona tulee olemaan kontrollerin hallinnoiman sivun osan "scope", eli Angular-magiaan kuuluva mystinen liima joka liittää näkymätemplatet (eli html-tiedostot) ja kontrollerit. 

Angular *injektoi* scopen automaattisesti kontrollerille, eli kun metodia kutsutaan, parametrilla on maagisesti oikea arvo. Jos parametri olisi nimetty eri tavalla, ei se saisi oikeaa arvoa. Angular siis antaa parametrille arvoksi nimenomaan scopen sen nimen <code>$scope</code> perusteella.

Palataan näkymätemplateen. Muutetaan sen sisältöä seuraavasti:

```html
      <h2>List here some stuff</h2>

      <ul>
        <li ng-repeat="thing in awesomeThings">
          {{thing}}     
        </li>
      </ul>
```

Käytössä on ehkä Angularin eniten käytetty *direktiivi* <code>ng-repeat</code>. Kuten arvata saattaa, direktiivi saa aikaan sen, että li-elementti monistuu siten että jokaiselle läpikäytävän kokoelman alkiolle muodostuu elementistä oma kopio.

Sensijaan että näyttäisimme sivulla listan kontrollerissa taulukkoon kovakoodattuja merkkijonoja, haluamme näyttää sovelluksessa internetistä, esim. sovelluksemme backendistä ladattavia asioita.

Törmäämme tässä kohdassa selainten rajoitteisiin: selain ei voi ladata vapaasti resursseja muista domainista kuin siitä miltä html-sivu on ladattu, ks. [http://en.wikipedia.org/wiki/Same_origin_policy](http://en.wikipedia.org/wiki/Same_origin_policy). Usein kuitenkin tilanne on se, että haluamme käyttää jotain ulkopuoleisia rajapintoja tai oma backendimme ei sijaitse samassa domainissa kuin mistä frontend-koodi haetaan.

Ratkaisuja ongelmaan on muutamia, käytämme näistä järkevintä eli
* [CORSia (Cross-Origin resource sharing)](http://en.wikipedia.org/wiki/Cross-Origin_Resource_Sharing)

Käytännössä CORS toteutetaan HTTP-headereiden avulla.

CORS edellyttää aina sitä että sovelluksen käyttämä backend tai palvelu on konfiguroitu sopivalla tavalla. Aina ei näin ole ja joudutaan käyttämään muita ratkaisuja esim JSONP:tä. 

Käytössämme oleva Rails-backend on konfiguroitu sallimaan CORS. Konfiguraatio on hyvin helppoa gemin [rack-cors](https://github.com/cyu/rack-cors) avulla.

Myös frontendin puolella tarvitaan pieni temppu.

Muutetaan sovelus-moduulin määrittelevä *main.js* seuraavaan muotoon:

```javascript
var app = angular.module('frontendApp', []);

app.config(function($httpProvider) {
    $httpProvider.defaults.useXDomain = true;
    delete $httpProvider.defaults.headers.common['X-Requested-With'];
});
```

Koodirivit muuttavat hieman sitä miten Angularin tarjoama <code>$http</code> service toimii oletusarvoisesti.

Muutetaan nyt sovelluksen kontrolleria seuraavasti:

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

Kontrollerille *injektoidaan* nyt scopen lisäksi myös <code>$http</code>, joka on Angularin valmiiksi tarjoama HTTP:n käytön abstraktoima palvelu, ks [https://docs.angularjs.org/api/ng/service/$http](https://docs.angularjs.org/api/ng/service/$http). 

Palvelun käyttö on helppoa. Kutsumme sen metodia get ja määrittelemme callback-fuktion
jota kutsutaan operaation onnistuessa. 

Olisimme voineet määritellä callbackin myös epäonnistuneeseen tapaukseen:

```javascript
    $http.get('http://osoite').success( function(data, status, headers, config) {

    }).error( function(data, status, headers, config) {

    });
```

Callback siis liittää palvelimen lähettämän datan scopeen asetettuun muuttujaan <code>entries</code>. Muutetaan näkymää siten, että se näyttää palvelimen palauttaman datan:

```javascript
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

Palvelin palauttaa siis taulukom json-muotoisia olioita joilla on mm. kentät subject, user ja body.

Tehdään sovellukseemme mahdollisuus blogien kirjoittamiseen. Luodaan ensin näkymään sopiva formi:

```html
      <h2>Create a new entry</h2>

      <form>
        name: <input type="text" ng-model="blog.user"></input>        
        <br>
        subject: <input type="text" ng-model="blog.subject"></input>
        <br>
        body:
        <input type="text" ng-model="blog.body"></input>
        <br>
        <button ng-submit="createBlog()">create</button>
      </form>

      {{blog}}
```

Lomake ei vielä tee mitään ja on ulkoasultaan ruma. Tarkastellaan kuitenkin paria asiaa. Jokaiseen kenttään on liitetty attribuuttidirektiivin *ng-model* avulla arvo, joka vastaa scopessa olevaa muuttujaa (tai sen kenttää) johon kentän arvo tallettuu.

Lomakkeen alle on lisätty rivi <code>{{blog}}</code> joka näyttää olion *blog* jonka kenttiin syötteen arvot sidotaan. Jos kokeilet kirjottaa jotain syötekenttiin, pääset todistamaan Angularin two way binding -magiaa.

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

Muutetaan callbackiä siten, että se luo blogin palvelimelle HTTP POST -kutsulla:

```javascript
    $scope.createBlog = function() {
    	$http.post('http://ng-project-backend.herokuapp.com/api/blogs.json', $scope.blog).success( function(data, status, headers, config) {
    		$scope.entries.push(data)
    	});
    	$scope.blog = {}
    }
```

Kutsu siis tehdään <code>$http</code>-palvelun avulla antamalla lähetettävä dataolio parametriksi. Onnistuneen tapauksen takaisinkutsussa laitetaan luotu blogikirjoitus blogien listalle jotta myös sivun käyttäjä näkee uuden blogin.

## omat servicet

Kontrollerin koodi on nyt sikäli ikävää että siihen on kovakoodattu backendin osoite. Kontrolleri myös käyttää suoraan $http-palvelua ja tämä hankaloittaa kontrollerin testaamista. 

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

Palvelut ovat käytännössä singleton-olioita joita on mahdollista *injektoida* esim. kontrollereihin.

Palvelujen määrittelytapoja on muutamia, käytämme seuraavassa tehdasmetodia:

```javascript
app.factory('Blogs', function($http){
    var URL = 'http://ng-project-backend.herokuapp.com/api/blogs.json'; 

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
```javascript

Kontrollerille on nyt injektoitava määrittelemämme <code>Blogs</code>-palvelu. Kontrollerilla ei ole enää riippuvuutta $http-palveluun, joten sitä ei tarvitse enää injektoida:

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

Koska käyttämämme Rails-resurssi noudattaa RESTFull-konventioita, olisimme myös voineet käyttää
$http-palvelun sijaan Angularin valmista
[$resource](https://docs.angularjs.org/api/ngResource/service/$resource)-palvelua jolloin olisimme selvinneet hivenen helpommalla. Tosin tässä tapauksessa palvelusta ei ole kovin suurta hyötyä koska aksessoimme _blogs_-resurssia vain kahda metodi/url-kombinaatiota käyttäen.

## hienosäätöä

Lomakkeemme on nyt koko ajan näkyvillä. Haluisimme sen aukeamaan vain tarvittaessa, esim. jotain painiketta klikkaamalla.

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
    	// piilota kun uusi blogi luotu
    	$scope.formVisible = false;
    	$scope.blog = {}
    }

});
```

## deployment

Nyt on korkea aika deployata sovellus herokuun. Toimenpide on helppo, ensimmäisen vaiheen ohje löytyy [täältä](http://www.lars-schenk.com/deploying-a-yeoman-angular-app-to-heroku/1661).

Toine vaihe on "kääntää" ohjelma komennolla <code>grunt build</code>. Tällöin ovelluksen käännetty ja js-koodin osalta minimoitu versio syntyy hakemistoon _dist_. Oletusarvoisesti _.gitigonre_ tiedosto ignoroi ko tiedoston, ignorointi tulee poistaa!

Seuraavaksi luodaan tavanomaiseen tapaan heroku-sovellus (komennolla <code>heroku create</code>) ja pushataan commitoitu repositorio (jossa siis on oltava mukana myös _dist_-hakemisto) herokuun. 

Sovelluksen pitäisi sitten toimia: [http://ng-project-fe.herokuapp.com](http://ng-project-fe.herokuapp.com)

## lisää...

```javascript
```

```javascript
```
