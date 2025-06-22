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

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

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

# BOO