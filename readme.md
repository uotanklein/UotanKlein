<div align="center">

  <img src="https://capsule-render.vercel.app/api?type=waving&height=230&color=0:0b1220,45:155e75,100:7c3aed&text=uotanklein&fontAlign=50&fontAlignY=37&fontColor=ffffff&fontSize=58&desc=Garry%27s%20Mod%20%2F%20Helix%20%2F%20Source%20Engine%20tooling&descAlign=50&descAlignY=58" alt="uotanklein banner" />

  <p>
    <a href="https://discord.gg/metro2033">
      <img src="https://img.shields.io/badge/Qanon%20Project-5865F2?style=for-the-badge&logo=discord&logoColor=white" alt="Qanon Project" />
    </a>
    <a href="https://t.me/uotanklein">
      <img src="https://img.shields.io/badge/Telegram-26A5E4?style=for-the-badge&logo=telegram&logoColor=white" alt="Telegram" />
    </a>
    <a href="mailto:francferdinant2@gmail.com">
      <img src="https://img.shields.io/badge/Email-EA4335?style=for-the-badge&logo=gmail&logoColor=white" alt="Email" />
    </a>
  </p>

</div>

## Ремарка перед контекстом

Сложно назвать это сухим профилем разработчика, ибо я в целом плохо верю в списки технологий без контекста. Тут обычно должен быть уверенный текст про passion, clean code и прочую витринную благодать, но в реальности всё куда прозаичнее: есть Source, есть GMod, есть внезапная хтонь, и кто-то должен разложить её по полочкам.

Поэтому начну не со стека. Стек - это лишь следствие. Сначала был проект.

## Сначала был Qanon

Сейчас моя основная территория - **Qanon Project**, большой Garry's Mod / Helix-гейммод. Это не один плагин и не набор случайных скриптов, а живой организм, где персонажи, предметы, оружие, зоны, радиация, болезни, навыки, крафт, экономика, интерфейсы и голос должны как-то сосуществовать и не убивать друг друга при следующем обновлении.

В вакууме любая из этих систем звучит просто. В реальном проекте всё начинает цепляться за всё: UI зависит от состояния, состояние утекает в сеть, сеть упирается в client/server/shared-разделение, а потом выясняется, что какая-нибудь сущность ведёт себя иначе не потому, что ты ошибся в логике, а потому что движок снова решил напомнить о своей природе.

И вот здесь у меня появляется привычка не просто писать фичу, а строить вокруг неё внутренний слой. Не из любви к абстракциям ради абстракций, а потому что без порядка GMod довольно быстро превращается в вязкую кашу. Сегодня ты добавил одну зону. Завтра у тебя уже `Builder`, `InterMngr`, кеши, сериализация и мысль: "А не пора ли это всё вынести в нормальную систему?"

## Потом приходит Source

Недавний пример: несколько зон генераторов с `apothem = 50000`. Казалось бы, просто большие trigger-объёмы. На деле AABB выходит за нормальные размеры карты, Source/Havok начинает вести себя как черный ящик, а зоны то работают, то нет. Пересоздание entity иногда оживляет случайные экземпляры, иногда не оживляет ничего, и ты сидишь ночью, пытаясь понять, это `StartTouch`, `EndTouch`, `core_entered_volume`, broad phase, или ты сам где-то сонный сотворил нечто непотребное.

Другой случай - декали на карте. `infodecal` и `info_projecteddecal` при старте могут временно создать edicts, забить лимит, а потом спокойно их освободить. Вот только постоянные сущности, которые должны были инициализироваться в этот момент, уже отпали. После старта лимит как будто свободен, но часть мира уже потеряна. Прекрасная, поучительная и немного издевательская специфика Source.

После таких ситуаций начинаешь ценить не красивые декларации, а явные ограничения, валидаторы, диагностику, asset tooling и системы, где хотя бы понятно, какой именно слой сейчас врет.

## Из этого вырос мой подход

Я довольно часто разделяю явления на отдельные сущности, даже если со стороны это выглядит как душнильство. Интерес к явлению не равен эмоциональной вовлеченности. Плохая фича не равна плохой идее. Кривая реализация не равна бесполезной системе. `VariableA != VariableB`, противоречие не обнаружено.

В коде логика та же. Состояние отдельно. Визуал отдельно. Сетевые побочные эффекты отдельно. Данные отдельно. Если запихнуть три разные переменные в одну константу, получится не архитектура, а очередной хтонический цирк, который потом придется чинить ночью.

Отсюда и мой набор привычек:

- держать фичу в общей конструкции, а не оставлять её одиночным костылем;
- заранее думать о client/server/shared-границе и о том, что уйдет в сеть;
- писать внутреннюю Lua-инфраструктуру, когда проект начинает повторять одни и те же паттерны;
- относиться к UI как к системе, а не как к декоративной прослойке поверх логики;
- автоматизировать ручную рутину, ибо повторяемая боль рано или поздно должна стать CLI, localhost-инструментом или хотя бы скриптом;
- уходить в низкий уровень, если обычного Lua или TypeScript уже недостаточно.

## Во что это вылилось

Так появились не только игровые системы, но и слой вокруг них: `classlib`, `Serializer`, `Promise/async`, `Builder`, `SoundController`, `InterMngr`, `MatInfo`, `Bitmap`. Некрасиво, но деконструкция мне доставляет. Мне нравится, когда у кода есть фундамент, мышцы, вода и техническая остаточность после использования.

Дальше та же логика вышла за пределы Lua. Если Workshop-аддоны неудобно скачивать и разбирать руками - появляется **GMDownloader**. Если контент карты нужно собирать, дедуплицировать и делить на части - появляется **Source Asset Toolkit**. Если TypeScript-инструменту нужен быстрый дочерний бинарник для трассировки `.mdl` и `.vmt` - появляется **source-asset-tracer** на Rust.

| Кейс | Суть | Стек |
| --- | --- | --- |
| **Qanon Project** | Большой Helix-гейммод с десятками плагинов и множеством связанных игровых систем | Lua, Helix, Derma |
| **Lua core layer** | Собственный внутренний слой: классы, сериализация, promise/async, builders, interaction/audio/material helpers | Lua, metatables, coroutine, net |
| **Voice & interaction systems** | Дистанция голоса, рации, PA-системы, 3D/2D voice FX, контроль взаимодействия с сущностями | Lua, Helix, GMod net |
| **ARC9 / weapons work** | Интеграция оружия с предметами, safe-state, spread/breath-модификаторы, переопределение поведения выстрелов | Lua, ARC9, SWEP |
| **GMDownloader** | Локальный web-инструмент для Workshop-аддонов: SteamCMD, gmad, file tree, просмотр и архивирование исходников | Next.js, React, TypeScript, PostgreSQL |
| **Source Asset Toolkit** | Упаковка VMF-ассетов, правила content-pack, дедупликация, split по размеру, SSE-progress, безопасная очистка output | TypeScript, Express, Node.js |
| **source-asset-tracer** | Дочерний Rust-бинарник для `vmf-asset-packer`, дабы трассировать `.mdl`/`.vmt` и собирать связанные `.vmt`/`.vtf`/model-файлы | Rust, keyvalues-parser, binary IO |
| **bsp-obfuscate** | Чтение BSP, parsing lumps, генерация LMP, эксперименты с устройством Source-карт | Bun, TypeScript, binary-parser |

## Стек, если всё же нужен список

<div align="center">

  <img src="https://skillicons.dev/icons?i=lua,ts,js,react,next,redux,nodejs,express,postgres,mysql,sqlite,python,rust,c,html,css,sass,tailwind,bootstrap,bun,vite,webpack,docker,git,github,npm,deno&perline=13" alt="Tech stack" />

</div>

## Если совсем коротко

Я не столько "full-stack developer", сколько человек, который пытается привести хаотичный игровой и контентный пайплайн к вменяемой форме. Иногда это Lua-плагин. Иногда web-инструмент. Иногда Rust-бинарник, который просто делает одну узкую вещь быстрее и надежнее.

В целом суть одна: меньше ручного ада, больше ясности в том, как система устроена. А если система всё равно развалилась - значит будет еще один слой деконструкции, пара ночных выводов и, вероятно, новый инструмент. Родная хтонь, что поделать.

## GitHub

<div align="center">

  <img height="170" src="https://github-readme-stats.vercel.app/api?username=uotanklein&show_icons=true&theme=tokyonight&hide_border=true&rank_icon=github" alt="GitHub stats" />
  <img height="170" src="https://github-readme-stats.vercel.app/api/top-langs/?username=uotanklein&hide=html,css&theme=tokyonight&hide_border=true&layout=compact" alt="Top languages" />

</div>

<div align="center">

  <img src="https://github-readme-streak-stats.herokuapp.com?user=uotanklein&theme=tokyonight&hide_border=true" alt="GitHub streak" />

</div>

## Контакты

<div align="center">

  <a href="https://discord.gg/metro2033">Qanon Project Discord</a>
  ·
  <a href="https://t.me/uotanklein">Telegram</a>
  ·
  <a href="mailto:francferdinant2@gmail.com">francferdinant2@gmail.com</a>

</div>

<img src="https://capsule-render.vercel.app/api?type=waving&height=120&section=footer&color=0:7c3aed,45:155e75,100:0b1220" alt="Footer" />
