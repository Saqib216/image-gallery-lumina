# 🖼️ Image Gallery: LUMINA

A dark-themed, fully responsive image gallery built with vanilla HTML, CSS, and JavaScript. Features category filtering, real-time search, a smooth lightbox viewer with keyboard navigation, and hover effects.

---

## 🔗 Live Demo

> _https://image-gallery-lumina.vercel.app_

---

## ✨ Features

- 12 curated high-quality photos across 4 categories
- Responsive CSS Grid layout — 3 columns → 2 → 1 on smaller screens
- Category filter buttons dynamically generated from image data
- Real-time search — filters by title as you type
- Combined filter + search logic working simultaneously
- Hover overlay with image title and category on each card
- Subtle zoom effect on card hover
- Lightbox viewer with full-screen backdrop blur
- Prev / Next navigation inside lightbox
- Image counter showing current position (e.g. 3 / 8)
- Keyboard support — Arrow keys to navigate, Escape to close
- Open Full Image button — opens full-resolution in a new tab
- Glassmorphism sticky header with backdrop blur
- Gradient hero heading — white to violet
- Custom accent-colored scrollbar
- Hover vs touch separation via `@media (hover: hover)` and `@media (hover: none)`

---

## 🗂️ Project Structure

```
ImageGallery_LUMINA/
├── index.html          # App markup — header, hero, filters, gallery grid, lightbox
├── css/
│   └── style.css       # Dark theme, glassmorphism, grid, hover effects, responsive
├── js/
│   └── script.js       # Image data array, render functions, filter/search, lightbox logic
└── README.md
```

---

## 🧠 JavaScript Logic: Deep Dive

All state lives in two top-level variables alongside the images array:

```js
let currentIndex = 0;
let activeImages = [];
```

`activeImages` always holds the currently filtered set — lightbox navigation works off this, not the full `images` array. This means prev/next in the lightbox respects whatever filter or search is active.

---

### `renderFilters()` — Dynamic Filter Buttons

Extracts unique categories from the `images` array using `map()` and `Set`, prepends "All", then loops to create `<button>` elements with `data-category` attributes:

```js
const categories = ["All", ...new Set(images.map(img => img.category))];
```

The "All" button gets an `active` class by default. No hardcoded category names anywhere — adding a new category to the data array automatically adds a new filter button.

---

### `renderGallery(filteredImages)` — Inject Cards into DOM

Clears `#gallery-grid`, then loops through `filteredImages` and creates each card using template literals:

```js
galleryGrid.innerHTML = "";
filteredImages.forEach((image, index) => {
    // creates .gallery-card with img + .card-overlay
    // attaches click listener → openLightbox(index)
});
```

The click listener is added here during render — each card knows its own index in the current filtered set, so lightbox navigation stays in sync with whatever is currently visible.

---

### `filterAndSearch()` — Combined Filter + Search

Reads two things simultaneously — the active filter button's `data-category` and the search input's current value — then filters the `images` array against both conditions at once:

```js
const activeBtn = filterButtons.querySelector(".filter-btn.active");
const activeBtnCategory = activeBtn.dataset.category;
const searchVal = searchInput.value.toLowerCase();

const filteredImages = images.filter(image => {
    return (activeBtnCategory === "All" || image.category === activeBtnCategory)
        && image.title.toLowerCase().includes(searchVal);
});

activeImages = filteredImages;
renderGallery(filteredImages);
```

`activeImages` is updated here so lightbox always navigates the filtered set, not all 12 images.

---

### Filter Button Event — Event Delegation

Instead of attaching listeners to each button individually, a single listener on the parent container handles all clicks:

```js
filterButtons.addEventListener("click", (e) => {
    if (e.target.classList.contains("filter-btn")) {
        filterButtons.querySelectorAll(".filter-btn").forEach(btn => btn.classList.remove("active"));
        e.target.classList.add("active");
        filterAndSearch();
    }
});
```

This works even after buttons are dynamically created — no timing issues.

---

### `openLightbox(index)` — Open at Clicked Image

Sets `currentIndex`, calls `updateLightbox()` to populate content, then adds `.active` to the lightbox element to make it visible via CSS:

```js
const openLightbox = (index) => {
    currentIndex = index;
    updateLightbox();
    lightbox.classList.add("active");
};
```

`.active` triggers `opacity: 1` and `visibility: visible` on the lightbox — the transition is handled entirely in CSS.

---

### `updateLightbox()` — Sync Content to Current Index

Single source of truth for lightbox content. Called by `openLightbox()` and `navigateLightbox()` — never duplicates logic:

```js
const updateLightbox = () => {
    lightboxImage.src = activeImages[currentIndex].src;
    lightboxTitle.textContent = activeImages[currentIndex].title;
    lightboxCategory.textContent = activeImages[currentIndex].category;
    currentIndexEl.textContent = currentIndex + 1;
    totalImagesEl.textContent = activeImages.length;
};
```

---

### `navigateLightbox(direction)` — Prev / Next with Wrapping

Takes `1` for next, `-1` for previous. Uses modulo to wrap around at both ends:

```js
currentIndex = (currentIndex + direction + activeImages.length) % activeImages.length;
updateLightbox();
```

At the last image, next wraps to index 0. At index 0, previous wraps to the last image.

---

### Keyboard Navigation

A single `keydown` listener handles all three keyboard interactions:

```js
document.addEventListener("keydown", (e) => {
    if (e.key === "Escape") closeLightbox();
    if (e.key === "ArrowLeft") navigateLightbox(-1);
    if (e.key === "ArrowRight") navigateLightbox(1);
});
```

---

## 🎨 Design Highlights

- **Color scheme:** `#0d0d0d` background, `#a78bfa` soft violet accent
- **Glassmorphism:** `backdrop-filter: blur(12px)` on sticky header and lightbox backdrop
- **Gradient hero text:** `linear-gradient(135deg, #ffffff, #a78bfa)` clipped to text using `-webkit-background-clip`
- **Card hover:** `.card-overlay` fades in from `opacity: 0` to `1`, image scales to `1.06` simultaneously
- **Hover vs touch:** All hover effects inside `@media (hover: hover)` — separate `:active` states inside `@media (hover: none)` handle touch devices
- **Aspect ratio:** `aspect-ratio: 4/3` keeps all cards uniform regardless of original image dimensions
- **Custom scrollbar:** 6px width, accent color on thumb hover, matches dark theme
- **Font:** Inter (Google Fonts) — clean and modern

---

## 🛠️ Tech Stack

| Tech | Usage |
|------|-------|
| HTML5 | Semantic structure |
| CSS3 | Dark theme, glassmorphism, grid layout, animations |
| JavaScript ES6+ | All gallery logic, DOM manipulation |
| Font Awesome 6.5 | Icons throughout UI |
| Inter (Google Fonts) | Typography |
| Unsplash | High-quality photo sources |
| Vercel | Deployment |

---

## 📱 Responsive Behavior

| Screen | Layout |
|--------|--------|
| Desktop (768px+) | 3-column grid, full filter row |
| Tablet (480px - 768px) | 2-column grid, filter controls stack vertically |
| Mobile (below 480px) | 1-column grid, lightbox nav stacks vertically |

---

## 🖼️ Image Categories

| Category | Count |
|---|---|
| Nature | 3 |
| Architecture | 3 |
| Travel | 3 |
| Abstract | 3 |

---

## 🔮 Future Enhancements

- Lazy loading images for better performance
- Drag / swipe support for lightbox on mobile
- LocalStorage — remember last active filter
- Download with fetch + blob when CORS allows
- Infinite scroll or pagination for larger collections
- Backend integration for dynamic image uploads

---

## 🚀 Getting Started

No build tools or dependencies required.

```bash
git clone https://github.com/Saqib216/image-gallery-lumina.git
cd ImageGallery
# Open index.html in your browser or use VS Code Live Server
```

---

## 👨‍💻 Author

**Muhammad Saqib Hussnain** 

- [GitHub](https://github.com/Saqib216)
- [LinkedIn](https://www.linkedin.com/in/saqib-hussnain)
- [Portfolio](https://saqib-portfo.netlify.app)
- [Instagram](https://instagram.com/itx.saqib.hussnain)

---

## 📄 License

This project is open source and available for educational and personal use.

---

*Built with vanilla HTML, CSS, and JavaScript. No frameworks. No shortcuts. Just code.*