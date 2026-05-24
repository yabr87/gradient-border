# Gradient Border

Два способи зробити градієнтний бордер на CSS.

**Демо:** https://yabr87.github.io/gradient-border/

---

## Метод 1 — background-clip

Найпростіший спосіб. Працює коли фон елемента — суцільний колір.

```css
.card-1 {
  border: 1px solid transparent;
  background-image:
    linear-gradient(#1e2122, #1e2122),               /* фон картки */
    linear-gradient(160deg, #23282a, #393f41, #23282a); /* градієнт бордера */
  background-origin: border-box;
  background-clip: padding-box, border-box;
  border-radius: 16px; /* необов'язково */
}
```

**Скорочена версія** — `background-origin` і `background-clip` можна об'єднати в одне слово прямо в `background`:

```css
.card-1 {
  border: 1px solid transparent;
  background:
    linear-gradient(#1e2122, #1e2122) padding-box,
    linear-gradient(160deg, #23282a, #393f41, #23282a) border-box;
  border-radius: 16px; /* необов'язково */
}
```

Одне ключове слово після градієнта одразу задає і `origin`, і `clip` для того шару.

**Як працює:** два фони — перший кліпається до `padding-box` (заповнює вміст), другий до `border-box` (заповнює бордер).

**Обмеження:** не працює з `background: transparent`.

---

## Метод 2 — ::before + mask (прозорий фон)

Підходить коли потрібен прозорий або blur-фон.

```css
.card-2 {
  position: relative;
  border-radius: 16px; /* необов'язково */
  z-index: 0; /* потрібен щоб ::before з z-index: -1 не пішов під body */
}

/* градієнтний бордер */
.card-2::before {
  content: '';
  position: absolute;
  inset: 0;
  border-radius: 16px; /* необов'язково, має збігатись з батьківським */
  padding: 1px; /* товщина бордера */
  background: linear-gradient(160deg, #23282a, #393f41, #23282a);
  -webkit-mask:
    linear-gradient(#fff 0 0) content-box,
    linear-gradient(#fff 0 0);
  -webkit-mask-composite: destination-out;
  mask-composite: exclude;
  z-index: -1;
}

/* blur-фон картки — необов'язковий блок, лише для glassmorphism-ефекту */
.card-2::after {
  content: '';
  position: absolute;
  inset: 0;
  border-radius: 16px; /* необов'язково */
  background: rgba(124, 134, 136, 0.06); /* необов'язково */
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px); /* необов'язково, для старих Safari */
  z-index: -2;
}
```

**Як працює:**

`::before` розтягується на весь елемент і заповнюється градієнтом. Маска вирізає центр — залишається тільки смужка по краю.

**Крок 1** — `position: absolute` + `inset: 0` розтягує псевдоелемент на весь батьківський блок.

**Крок 2** — `padding: 1px` задає товщину бордера. Це створює два шари площі: зовнішній (`border-box`) і внутрішній (`content-box`). Різниця між ними — і є бордер шириною 1px.

**Крок 3** — `background: linear-gradient(...)` заповнює градієнтом весь псевдоелемент.

**Крок 4** — маска вирізає центр:
```css
-webkit-mask:
  linear-gradient(#fff 0 0) content-box,  /* маска на внутрішню зону */
  linear-gradient(#fff 0 0);              /* маска на весь елемент */
mask-composite: exclude;
```
`exclude` віднімає першу маску від другої — видимою залишається тільки зона `padding`, тобто смужка 1px по краю.

```
┌─────────────────────┐
│ ░░░░░░░░░░░░░░░░░░░ │  ← border-box (градієнт є)
│ ░┌─────────────┐░░ │
│ ░│             │░░ │  ← content-box вирізається маскою
│ ░└─────────────┘░░ │
│ ░░░░░░░░░░░░░░░░░░░ │
└─────────────────────┘
   ↑ залишається тільки ця смужка = градієнтний бордер
```
