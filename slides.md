---
theme: seriph
background: https://cover.sli.dev
title: Eloquent vs. GTFS - O Quebra-cabeças da Chave Primária Composta
class: text-center
drawings:
  persist: false
transition: slide-left
mdc: true
---

# Eloquent vs. GTFS

O Quebra-cabeças da Chave Primária Composta

<div @click="$slidev.nav.next" class="mt-12 py-1">
<p>André São Pedro - Full Stack Developer (Junior Consultant)</p>
<p>Latitudde - a CONKORD Company</p>
</div>

<div class="abs-br m-6 text-xl">
  <a href="https://www.linkedin.com/in/afbsaopedro" target="_blank" class="slidev-icon-btn">
    <carbon:logo-linkedin />
  </a>
  <a href="https://github.com/afbsaopedro" target="_blank" class="slidev-icon-btn">
    <carbon:logo-github />
  </a>
</div>

---
transition: fade-out
---

# Quando _standards_ colidem

- GTFS entende que algumas das suas tabelas tem chaves primárias compostas 🔑
- ORM do Laravel, Eloquent ORM, não tem suporte para chaves primárias compostas ❌

````md magic-move {lines: true}
```php {all}{lines:true}
// O que preciso de escrever:
class StopTime extends Model {
    protected $primaryKey = ['trip_id', 'stop_id', 'stop_sequence'];
    // ❌ Não suportado em Eloquent
}
```

```php {all}{lines:true}
// O que é suportado:
class StopTime extends Model {
    protected $primaryKey = 'trip_id';
    // 😢 no array me sad
}
```
````
<br>
<div style="display: flex; justify-content: center; align-items: center; gap: 20px;">
  <img src="./images/gtfs-qr.jpeg" alt="GTFS" style="width: 200px;">
  <img src="./images/laravel-qr.png" alt="Laravel" style="width: 200px;">
</div>  

<style>
h1 {
  background-color: #2B90B6;
  background-size: 100%;
  background: linear-gradient(80deg, #f61500 10%, #fefc41 40%);
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---
transition: fade-out
---

# Como solucionar o problema?

````md magic-move {lines: true}
```php {*|1,6-7,12-14,19-20}{lines:true}
// Como passar de algo limpo ✨:
class StopController extends Controller
{
    public function index()
    {
      // Linha única a utilizar o Model 👌
        return view('gtfs.stops.index', ['stops' => Stop::all()]);
    }

    public function store(StoreStopRequest $request): RedirectResponse
    {
      // Linhas mínimas a utilizar o Model 🤌
        Stop::create($request->except('_token'));
        return redirect(route('gtfs.stops.index'))->with('success', 'Paragem criada com sucesso!');
    }

    public function show(string $id): View
    {
      // Linha única a utilizar o Model e um Id normal 🫧
        return view('gtfs.stops.show', ['stop' => Stop::findOrFail($id)]);
    }
}
```

```php {*|1-3,8-14}{lines:true}
// Without making a mess 🤮
// Queremos evitar linhas e lógica que não sejam receber ou enviar dados no Controller
// Better readability and easier to understand 📖
class CalendarDateController extends Controller
{
    public function index(): View
    {
      // Lógica necessária para encontrar todos os CalendarDates na versão atual 🤢
      $calendarDates = CalendarDate::all()
      ->where('version_id', GtfsVersioningService::getCurrentVersion()->version_id);

        return view('gtfs.calendar_dates.index', [
            'calendarDates' => $calendarDates
        ]);
    }
}
```

```php {5-12}{lines:true}
class CalendarDateController extends Controller
{
    public function store(StoreCalendarDateRequest $request): RedirectResponse
    {
      // Lógica necessária para preencher todos os campos necessários que possibilitam
      // o sistema de versionamento 😨
      $data = $request->except('_token')  
      $data['version_id'] = GtfsVersioningService::getCurrentVersion()->version_id;
      $data['service_version_id'] = GtfsVersioningService::getCurrentVersion()->version_id;
      CalendarDate::create($data);
      return redirect(route('gtfs.calendar_dates.index'));
    }
}
```

```php {5-10}{lines:true}
class CalendarDateController extends Controller
{
    public function show(string $serviceId, $date): View
    {
      // Sem chave primária composta, somos obrigados a construir uma query
      // Temos ainda a agravante das versões 🙃
      $calendarDate = CalendarDate::where('version_id', GtfsVersioningService::getCurrentVersion()->version_id)
            ->where('date', $date)
            ->where('service_id', $serviceId)
            ->firstOrFail();

        return view('gtfs.calendar_dates.show', ['calendarDate' => $calendarDate]);
    } 
}
```
````

---
transition: fade-out
---

# Repository Pattern to the rescue!

````md magic-move {lines: true}
```php {5-11}{lines:true}
class CalendarDateController extends Controller
{
    public function index(): View
    {
      // Bruh 💢
      $calendarDates = CalendarDate::all()
      ->where('version_id', GtfsVersioningService::getCurrentVersion()->version_id);

        return view('gtfs.calendar_dates.index', [
            'calendarDates' => $calendarDates
        ]);
    }
}
```

```php {5-8}{lines:true}
class CalendarDateController extends Controller
{
    public function index(): View
    {
      // Agora sim 😊
        return view('gtfs.calendar_dates.index', [
            'calendarDates' => CalendarDateRepository::all()
        ]);
    }
}
```


```php {5-10}{lines:true}
class CalendarDateController extends Controller
{
    public function store(StoreCalendarDateRequest $request): RedirectResponse
    {
      // Bruh 💢
      $data = $request->except('_token')  
      $data['version_id'] = GtfsVersioningService::getCurrentVersion()->version_id;
      $data['service_version_id'] = GtfsVersioningService::getCurrentVersion()->version_id;
      CalendarDate::create($data);
      return redirect(route('gtfs.calendar_dates.index'));
    }
}
```

```php {5-7}{lines:true}
class CalendarDateController extends Controller
{
    public function store(StoreCalendarDateRequest $request): RedirectResponse
    {
      // Agora sim 😊
        CalendarDateRepository::store($request->except('_token'));
        return redirect(route('gtfs.calendar_dates.index'));
    }
}
```

```php {5-11}{lines:true}
class CalendarDateController extends Controller
{
    public function show(string $serviceId, $date): View
    {
      // Bruh 💢
      $calendarDate = CalendarDate::where('version_id', GtfsVersioningService::getCurrentVersion()->version_id)
            ->where('date', $date)
            ->where('service_id', $serviceId)
            ->firstOrFail();

        return view('gtfs.calendar_dates.show', ['calendarDate' => $calendarDate]);
    } 
}
```

```php {5-8}{lines:true}
class CalendarDateController extends Controller
{
    public function show(string $serviceId, $date): View
    {
      // Agora sim 😊
        return view('gtfs.calendar_dates.show', [
          'calendarDate' => CalendarDateRepository::findOrFail($serviceId, $date)
          ]);
    }
}
```
````

---
transition: fade-out
---

# E as chaves compostas???

Usando o CalendarDate como exemplo:

````md magic-move {lines: true}
```php {*|3-5|7-11}{lines:true}
class CalendarDateRepository
{
  // Temos sempre de apanhar todos os campos utilizados na nossa chave primária composta 🧮
  // Neste caso $serviceId e $date, que nos são enviado do request vindo recebido pelo Controller
    public static function findOrFail(string $serviceId, string $date)
    {
      // Criar uma query através do modelo em questão
        return CalendarDate::where('version_id', GtfsVersioningService::getCurrentVersion()->version_id)
            ->where('date', $date)
            ->where('service_id', $serviceId)
            ->firstOrFail();
    }
}
```

```php {3-10|7-9}{lines:true}
class CalendarDateRepository
{
  // Podemos ainda criar toda a lógica que precisamos dentro do nosso Repository 
  // Assim evitamos ter a lógica espalhada em locais menos pertinentes 🤔
    public static function prepareData(array $data): array
    {
        $data['version_id'] = GtfsVersioningService::getCurrentVersion()->version_id;
        $data['service_version_id'] = GtfsVersioningService::getCurrentVersion()->version_id;
        return $data;
    }
}
```

```php {3-7}{lines:true}
class CalendarDateRepository
{
  // Exemplo a utilizar o prepareData dentro do repositório para atualizar um registo 😲
    public static function update($serviceId, $date, $data) {
        $data = self::prepareData($data);
        self::findOrFail($serviceId, $date)->update($data);
    }
}
```
````

---
transition: fade-out
---

# The End lol

I will NOT be taking any question ❓