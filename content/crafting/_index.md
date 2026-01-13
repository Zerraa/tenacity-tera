---
title: Crafting Calculator [TEST]
description: 
weight: 1
toc: true
bodyClass: wide-page
index: false
---

<link rel="stylesheet" href="/css/crafting.css">

<div class="page-layout">

  <!-- LEFT COLUMN -->
  <div class="left-column">

  <div class="left-split">

  <!-- BUFFS COLUMN -->
  <div class="buff-column">
      <div class="totals-sidebar buff-panel">
        <center><h3>Craft Buffs</h3></center>
        <div id="buff-options"></div>
        <div class="buff-results" id="buff-results">
          <p>Select an item to see crit info.</p>
    </div>
  </div>
  </div>
  

  <!-- MAIN COLUMN -->
  <div class="main-column">
      <div class="search-box">
        <button id="favorite-btn" onclick="toggleFavorite()" style="
    margin-left: 5px;
    padding: 8px 8px;
    font-size: 1em;
    border-radius: 4px;
    border: 1px solid #444;
    background-color: #222;
    color: #ffd479;
    width: 42px;
    height: 42px;
">☆</button>
        <input id="itemInput" placeholder="Start typing..." oninput="showSuggestions()" autocomplete="off" />
        <input id="itemQty" type="number" value="1" min="1" max="9999" style="width:90px; text-align:right; padding:6px 8px; font-size:1em; margin-left:5px; margin-right:10px;" />
        <div id="suggestions"></div>
        <button onclick="calculate()">Show Materials</button>
        <button onclick="openClearModal()">Clear</button>
      </div>

  <div id="results" style="margin-top:10px;"></div>
  <center><div id="error-message" style="color:red; margin-bottom:10px;"></div>
  </div></center>

  </div>

</div>


  <!-- RIGHT COLUMN -->
  <div id="totals-wrapper" class="totals-sidebar" style="position:relative;">
  <center><h3>Totals</h3></center>
  <ul id="totals-list"></ul>
</div>


</div>
 

<div id="bottom-bar" class="bottom-bar">
<hr>
  <div class="bottom-row">
    <strong>Favorites:</strong>
    <span id="favorites-list"></span>
  </div>
  <div class="bottom-row">
    <strong>Last Viewed:</strong>
    <span id="last-viewed-list"></span>
  </div>
</div>




<div id="floating-tooltip" style="
    position:absolute;
    display:none;
    background:#222;
    color:#fff;
    padding:8px;
    border-radius:6px;
    max-width:300px;
    z-index:9999;
    box-shadow:0 0 10px rgba(0,0,0,0.6);
    font-size:0.9em;
"></div>

<!-- Clear Confirmation Modal -->
<div id="clear-modal" style="
    display: none;
    position: fixed;
    top: 0; left: 0; right:0; bottom:0;
    background: rgba(0,0,0,0.7);
    z-index: 10000;
    align-items: center;
    justify-content: center;
    opacity: 0;
    transition: opacity 0.3s ease;
">
  <div style="
      background: #1e1e1e;
      color: #eee;
      padding: 20px 25px;
      border-radius: 10px;
      max-width: 400px;
      width: 90%;
      box-shadow: 0 0 20px rgba(0,0,0,0.6);
      transform: translateY(-20px);
      transition: transform 0.3s ease;
      text-align: center;
  ">
    <p style="margin-bottom: 15px; font-size: 1em; font-weight:500;">
      What do you want to clear?
    </p>
    <div style="display:flex; flex-direction: column; align-items: flex-start; gap: 12px; margin-bottom: 20px;">
      <label style="cursor:pointer; display:flex; align-items:center; gap:10px;">
        <input type="checkbox" id="clear-favorites" style="accent-color:#ffd479; width:18px; height:18px; display:block; margin:0;">
        <span style="line-height:18px;">Favorites</span>
      </label>
      <label style="cursor:pointer; display:flex; align-items:center; gap:10px;">
        <input type="checkbox" id="clear-lastviewed" style="accent-color:#ffd479; width:18px; height:18px; display:block; margin:0;">
        <span style="line-height:18px;">Last Viewed</span>
      </label>
    </div>
    <div style="display:flex; justify-content: center; gap: 15px;">
      <button id="clear-yes" style="
          padding: 8px 16px; 
          border-radius:6px;
          border:none;
          background-color:#444;
          color:#ffd479;
          font-weight:500;
          cursor:pointer;
          transition: background 0.2s;
      " onmouseover="this.style.backgroundColor='#555'" onmouseout="this.style.backgroundColor='#444'">
        Clear
      </button>
      <button id="clear-no" style="
          padding: 8px 16px; 
          border-radius:6px;
          border:none;
          background-color:#333;
          color:#aaa;
          font-weight:500;
          cursor:pointer;
          transition: background 0.2s;
      " onmouseover="this.style.backgroundColor='#444'" onmouseout="this.style.backgroundColor='#333'">
        Cancel
      </button>
    </div>
  </div>
</div>

<div id="toast" class="toast">Crafting copied ✓</div>



<script>

const AUTO_LOAD_LAST_RECIPE = false;

function resetCalculator() {
  document.getElementById("itemInput").value = "";
  document.getElementById("itemQty").value = 1;
  document.getElementById("results").innerHTML = "";
  document.getElementById("totals-list").innerHTML = "";
  document.getElementById("buff-results").innerHTML = "<p>Select an item to see crit info.</p>";

  for (const key in craftBuffs) {
    const buff = craftBuffs[key];
    if (buff.options) {
      const select = document.getElementById(`buff-${key}`);
      if (select) select.selectedIndex = 0;
    } else {
      const checkbox = document.getElementById(`buff-${key}`);
      if (checkbox) checkbox.checked = false;
    }
  }

  localStorage.setItem("lastViewedCleared", "1"); // ✅ important
}

function confirmClear() {
  // Ask if user wants to clear Last Viewed as well
  const clearLastViewed = confirm(
    "Do you want to also clear Last Viewed Recipes?\n\nOK = Clear Last Viewed\nCancel = Keep Last Viewed"
  );

  // Call existing reset logic
  resetCalculator();

  // If confirmed, clear last viewed from localStorage
  if (clearLastViewed) {
    localStorage.removeItem("lastViewedRecipes");
    renderLastViewedRecipes(); // refresh bottom bar
  }
}

function openClearModal() {
  const modal = document.getElementById("clear-modal");
  modal.style.display = "flex";

  // Reset checkboxes
  document.getElementById("clear-favorites").checked = false;
  document.getElementById("clear-lastviewed").checked = false;

  // Small timeout to trigger transition
  setTimeout(() => {
    modal.style.opacity = 1;
    modal.querySelector('div').style.transform = "translateY(0)";
  }, 10);
}

function closeClearModal() {
  const modal = document.getElementById("clear-modal");
  modal.style.opacity = 0;
  modal.querySelector('div').style.transform = "translateY(-20px)";
  setTimeout(() => {
    modal.style.display = "none";
  }, 300);
}

document.getElementById("clear-yes").addEventListener("click", () => {
  const clearFavs = document.getElementById("clear-favorites").checked;
  const clearLast = document.getElementById("clear-lastviewed").checked;

  // ✅ Always reset calculator (totals, buffs, item input)
  resetCalculator();

  // Only clear favorites if checkbox checked
  if (clearFavs) {
    localStorage.removeItem("favoriteRecipes");
  }

  // Only clear last viewed if checkbox checked
  if (clearLast) {
    localStorage.removeItem("lastViewedRecipes");
  }

  // Re-render lists after clearing
  renderFavorites();
  renderLastViewedRecipes();

  // Close modal
  document.getElementById("clear-modal").style.display = "none";
});


document.getElementById("clear-no").addEventListener("click", () => {
  closeClearModal();
});

function getFavorites() {
  return JSON.parse(localStorage.getItem("favoriteRecipes") || "[]");
}

function saveFavorites(favorites) {
  localStorage.setItem("favoriteRecipes", JSON.stringify(favorites));
}

function toggleFavorite() {
  const item = document.getElementById("itemInput").value.trim();
  if (!item) return;
  const qty = parseInt(document.getElementById("itemQty").value, 10) || 1;

  let favorites = getFavorites();
  const existingIndex = favorites.findIndex(f => f.name === item);

  if (existingIndex >= 0) {
    favorites.splice(existingIndex, 1); // remove
  } else {
    favorites.unshift({ name: item, qty });
    if (favorites.length > 10) favorites = favorites.slice(0, 10);
  }

  saveFavorites(favorites);
  renderFavorites();
  updateFavoriteButton();
}


function renderFavorites() {
  const favorites = getFavorites();
  const container = document.getElementById("favorites-list");
  if (!container) return;

  container.innerHTML = ""; // clear previous

  favorites.forEach(fav => {
    const link = document.createElement("a");
    link.href = "#";
    link.textContent = fav.name;
    link.style.marginRight = "8px";
    link.style.color = "#ffd479";

    link.addEventListener("click", e => {
      e.preventDefault();
      selectFavoriteFromObj(fav); // pass object directly
    });

    container.appendChild(link);
  });
}


function selectFavoriteFromObj(obj) {
  document.getElementById("itemInput").value = obj.name;
  document.getElementById("itemQty").value = obj.qty || 1;

  // ✅ Always calculate, which adds it to last viewed if not a favorite
  calculate();
  updateFavoriteButton();
  hideSuggestions(); // ✅ hide suggestions
}



function updateFavoriteButton() {
  const item = document.getElementById("itemInput").value.trim();
  const favorites = getFavorites();
  const btn = document.getElementById("favorite-btn");
  if (!btn) return;
  btn.textContent = favorites.some(f => f.name === item) ? "★" : "☆";
}



// Auto-render favorites on page load
window.addEventListener("DOMContentLoaded", () => {
  renderFavorites();
});



// --- Modify DOMContentLoaded to respect this flag ---
window.addEventListener("DOMContentLoaded", () => {
  renderLastViewedRecipes();

  const cleared = localStorage.getItem("lastViewedCleared");
  const lastRecipe = JSON.parse(localStorage.getItem("lastViewedRecipes") || "[]")[0];

  if (!cleared && lastRecipe && recipes[lastRecipe]) {
    document.getElementById("itemInput").value = lastRecipe;
    calculate();
  }
});



function addLastViewedRecipe(name) {
  if (!recipes[name]) return; // skip invalid recipes

  const favorites = getFavorites(); // get current favorites
  // ✅ FIX: check by name, not object
  if (favorites.some(f => f.name === name)) return;

  let lastViewed = JSON.parse(localStorage.getItem("lastViewedRecipes") || "[]");

  lastViewed = lastViewed.filter(r => r !== name); // remove duplicates
  lastViewed.unshift(name); // add to front

  if (lastViewed.length > 3) lastViewed = lastViewed.slice(0, 3);
  localStorage.setItem("lastViewedRecipes", JSON.stringify(lastViewed));

  renderLastViewedRecipes();
}


function renderLastViewedRecipes() {
  const lastViewed = JSON.parse(localStorage.getItem("lastViewedRecipes") || "[]");
  const container = document.getElementById("last-viewed-list");

  if (!container) return;

  container.innerHTML = lastViewed
    .map(name => {
  const safeName = name.replace(/'/g, "\\'");
  return `<a href="#" onclick="selectLastViewed('${safeName}'); return false;" style="margin-right:8px;color:#ffd479;">${name}</a>`;
})
    .join("");
}

function selectLastViewed(name) {
  document.getElementById("itemInput").value = name;
  calculate();
  hideSuggestions(); // ✅ hide suggestions
}


const craftBuffs = {
  food1: { label: "Soft Omelet", time: 0.2 },                
  food2: { label: "Sweet Pudding", crit: 0.15 },    
  gear: { label: "Lifeskiller's Armor",  crit: 0.05, time: 0.05 },            
  pet: {
    label: "Pet Buff",
    options: [
      { name: "No Pet Buff", crit: 0, time: 0 },                       
      { name: "Pet: Swifty Nugget", time: 0.2 },
      { name: "Pet: Lucky Nugget", crit: 0.2 }
    ]
  },
  craftAdditive: {
    label: "Additive",
    options: [
      { name: "No Additive", crit: 0 },                       
      { name: "I", crit: 0.025},                
      { name: "II", crit: 0.05},
      { name: "III", crit: 0.1},
      { name: "IV", crit: 0.15},
      { name: "V", crit: 0.2},
      { name: "Sweet Cooking Honey", crit: 0.10 }            
    ]
  },
  Server: { label: "Server Buff", crit: 0.05, time: 0.05 },  
  GM:     { label: "GM Buff",     crit: 0.2, time: 0.2 },
  Event: { label: "Event Buff", crit: 0.1, time: 0.1 }, 
};


let recipesLoaded = false;
let recipes = {};
let dataLoaded = {
  recipes: false,
  prices: false,
  water: false
};
let prices = {};
let taxPercent = 5; // example: 5% tax
let baseMaterials = {};
let tooltips = {};
let itemDurations = {};

fetch("/data/tooltips.json")
  .then(res => res.json())
  .then(data => tooltips = data);

fetch("/data/recipes.json")
  .then(res => res.json())
  .then(data => {
    recipes = data;
    dataLoaded.recipes = true;
    renderBuffOptions();
    tryAutoLoadLastRecipe();
  });

fetch("/data/prices.json")
  .then(res => res.json())
  .then(data => {
    prices = data;
    dataLoaded.prices = true;
    tryAutoLoadLastRecipe();
  });

fetch("/data/base_material.json")
  .then(res => res.json())
  .then(data => {
    baseMaterials = data;
    dataLoaded.water = true;
    tryAutoLoadLastRecipe();
  });

fetch("/data/itemDurations.json")
  .then(res => res.json())
  .then(data => itemDurations = data);

function tryAutoLoadLastRecipe() {
  if (!AUTO_LOAD_LAST_RECIPE) return; // ✅ hard stop

  if (!dataLoaded.recipes || !dataLoaded.prices || !dataLoaded.water) return;

  const lastRecipe = JSON.parse(localStorage.getItem("lastViewedRecipes") || "[]")[0];
  if (lastRecipe && recipes[lastRecipe]) {
    document.getElementById("itemInput").value = lastRecipe;
    calculate();
  }
}


// --- RENDER MATERIAL TREE ---
let nodeId = 0;

function toggleNode(id, btn) {
  const el = document.getElementById(id);
  const isHidden = el.classList.toggle("hidden");
  btn.textContent = isHidden ? "+" : "−";
}


let lastRecipeRendered = null;

function calculate() {
  const item = document.getElementById("itemInput").value.trim();
  const errorBox = document.getElementById("error-message");
  errorBox.textContent = ""; // clear previous error

  if (!item || !recipes[item]) {
    // Show error
    errorBox.textContent = "⚠ Please select a valid item.";
    
    // Clear outputs
    document.getElementById("results").innerHTML = "";
    document.getElementById("totals-list").innerHTML = "";
    document.getElementById("buff-results").innerHTML = "<p>Select an item to see crit info.</p>";

    return; // stop
  }

  // Clear error if valid
  errorBox.textContent = "";

  // Continue with normal calculation...
  addLastViewedRecipe(item);
  updateFavoriteButton();
  localStorage.removeItem("lastViewedCleared");

  const qty = parseInt(document.getElementById("itemQty").value, 10) || 1;
  nodeId = 0;

  const totals = {};
  lastCalculatedTotals = totals;

  const resultsHTML = renderMaterials(item, qty, 0, totals);
  document.getElementById("results").innerHTML = resultsHTML;

  for (const name in totals) {
    if (baseMaterials[name]) {
      const { baseItem, amount } = baseMaterials[name];
      const unitPrice = prices[baseItem] || 0;
      totals[name + "_basePrice"] = (totals[name + "_basePrice"] || 0) + totals[name] * amount;
    }
  }

  updateTotals(totals);
  updateBuffResults();

  if (lastRecipeRendered !== item) {
    renderBuffOptions();
    lastRecipeRendered = item;
  }
}



let qtyDebounce;
document.getElementById("itemQty").addEventListener("input", () => {
  clearTimeout(qtyDebounce);
  qtyDebounce = setTimeout(() => {
    const item = document.getElementById("itemInput").value.trim();
    if (!item || !recipes[item]) return;

    const qty = parseInt(document.getElementById("itemQty").value, 10) || 1;

    nodeId = 0;
    const totals = {}; // ✅ fresh totals object

    // Pass totals to renderMaterials
    document.getElementById("results").innerHTML = renderMaterials(item, qty, 0, totals);

    updateTotals(totals);   // ✅ pass the totals object
    updateBuffResults();    // recalc crit / time

    // ❌ Don't re-render buffs here; keep current selections
    // renderBuffOptions(); 
  }, 100); // 100ms debounce
});


function updateTotals(totals) {
  const totalsList = document.getElementById("totals-list");
  if (!totals || Object.keys(totals).length === 0) {
    totalsList.innerHTML = "";
    return;
  }

  let grandTotal = 0;
  totalsList.innerHTML = Object.entries(totals)
    .filter(([name]) => !name.endsWith("_basePrice"))
    .map(([name, amt]) => {
      let html = "";
      let cost = 0;

      if (baseMaterials[name]) {
        const { baseItem, amount } = baseMaterials[name];
        const unitPrice = prices[baseItem] || 0;
        const totalBase = amt * amount;
        const totalCost = unitPrice ? Math.floor(unitPrice * totalBase * (1 + taxPercent / 100)) : 0;
        if (totalCost) grandTotal += totalCost;

        html += `
          <li>
            <span>${name} × ${formatNumberEU(amt)}</span>
          </li>
          <li class="nested">
            <span>→ ${baseItem} × ${formatNumberEU(totalBase)}</span>
            ${unitPrice ? `<span class="unit">@ ${formatMoney(unitPrice)}</span>` : ""}
            ${totalCost ? `<span class="cost">${formatMoney(totalCost)}</span>` : ""}
          </li>
        `;
      } else {
        const unitPrice = prices[name] || 0;
        cost = unitPrice ? Math.floor(unitPrice * amt * (1 + taxPercent / 100)) : 0;
        if (cost) grandTotal += cost;

        html += `
          <li>
            <span>
              ${name} × ${formatNumberEU(amt)}
              ${unitPrice ? `<span class="unit">@ ${formatMoney(unitPrice)}</span>` : ""}
            </span>
            ${cost ? `<span class="cost">${formatMoney(cost)}</span>` : ""}
          </li>
        `;
      }

      return html;
    })
    .join("");

  if (grandTotal > 0) {
    totalsList.innerHTML += `
      <li class="grand-total">
        <span>Total Cost</span>
        <span class="cost">${formatMoney(grandTotal)}</span>
      </li>
    `;
  }
}


function formatMoney(bronze) {
  bronze = Math.floor(Number(bronze) || 0);

  const gold = Math.floor(bronze / 10000);
  bronze %= 10000;

  const silver = Math.floor(bronze / 100);
  bronze %= 100;

  const parts = [];
  if (gold) parts.push(`${gold.toLocaleString("de-DE")}g`);
  if (silver) parts.push(`${silver}s`);
  if (bronze || parts.length === 0) parts.push(`${bronze}b`);

  return parts.join(" ");
}


function formatNumberEU(num) {
  return Number(num).toLocaleString("de-DE");
}

let selectedIndex = -1; // tracks highlighted suggestion

function showSuggestions() {
  const input = itemInput.value.trim().toLowerCase();
  suggestionsDiv.innerHTML = "";
  errorBox.style.display = "none";
  selectedIndex = -1;

  if (!input) {
    suggestionsDiv.style.display = "none";
    return;
  }

  const suffixPriority = { "": 0, "Special": 1, "Marvelous": 2 };

  const matches = Object.keys(recipes)
    .filter(name => name.toLowerCase().includes(input))
    .sort((a, b) => {
      const aLower = a.toLowerCase();
      const bLower = b.toLowerCase();

      // prioritize items starting with input
      const aStarts = aLower.startsWith(input) ? 0 : 1;
      const bStarts = bLower.startsWith(input) ? 0 : 1;
      if (aStarts !== bStarts) return aStarts - bStarts;

      // prioritize by suffix
      const getSuffix = (str) => {
        if (str.startsWith("Marvelous")) return "Marvelous";
        if (str.startsWith("Special")) return "Special";
        return "";
      };
      const aSuffix = suffixPriority[getSuffix(a)] ?? 99;
      const bSuffix = suffixPriority[getSuffix(b)] ?? 99;
      if (aSuffix !== bSuffix) return aSuffix - bSuffix;

      // fallback alphabetical
      return aLower.localeCompare(bLower);
    });

  if (matches.length === 0) {
    suggestionsDiv.style.display = "none";
    return;
  }

  suggestionsDiv.style.display = "block";

  matches.forEach((name, i) => {
    const div = document.createElement("div");
    div.textContent = name;
    div.style.cursor = "pointer";
    div.style.padding = "6px";

    // hover highlight
    div.addEventListener("mouseenter", () => {
      setHighlight(i);
    });

    // click selection
    div.addEventListener("mousedown", () => {
      selectSuggestion(name);
    });

    suggestionsDiv.appendChild(div);
  });
}


function selectSuggestion(name) {
  document.getElementById("itemInput").value = name;
  document.getElementById("suggestions").style.display = "none";
  itemInput.value = name;
  suggestionsDiv.style.display = "none";
  errorBox.style.display = "none";
  calculate();

  // Save last viewed recipe
  localStorage.setItem("lastViewedRecipe", name);
  addLastViewedRecipe(name);
  hideSuggestions(); // ✅ hide suggestions
}

window.addEventListener("DOMContentLoaded", () => {
  renderFavorites();
  renderLastViewedRecipes();
});

function renderTotals(item) {
  const totals = calculateTotals(item);
  const totalsList = document.getElementById("totals-list");
  totalsList.innerHTML = Object.entries(totals)
    .map(([name, amt]) => `<li>${name} × ${amt}</li>`)
    .join('');
}

let totals = {};

function renderMaterials(item, multiplier = 1, depth = 0, totals) {
    const id = `node-${nodeId++}`;
    const indent = depth * 20;
    const isRoot = depth === 0;

    const info = tooltips[item] || {};
    const rawTooltip = info.tooltip || "";
    const tooltipText = formatTooltip(rawTooltip).replace(/"/g, "&quot;");
    const rarityClass = info.rarity ? `rarity-${info.rarity}` : "";

    if (!recipes[item]) {
        totals[item] = (totals[item] || 0) + multiplier;
        return `
        <div class="material-row ${rarityClass}" style="margin-left:${indent}px">
            <span class="material-text" data-tooltip="${tooltipText}">
                • ${item} × ${multiplier}
            </span>
        </div>
        `;
    }

    let html = `
    <div class="material-row ${rarityClass}" style="margin-left:${indent}px">
        <button class="toggle-btn" onclick="toggleNode('${id}', this)">
            ${isRoot ? "−" : "+"}
        </button>
        <span class="material-text" data-tooltip="${tooltipText}">
            <strong>${item}</strong> × ${multiplier}
        </span>
    </div>
    <div id="${id}" class="material-children ${isRoot ? "" : "hidden"}">
    `;

    for (const [mat, amt] of Object.entries(recipes[item].materials)) {
        html += renderMaterials(mat, amt * multiplier, depth + 1, totals);
    }

    html += `</div>`;
    return html;
}



const tooltipEl = document.getElementById("floating-tooltip");

document.addEventListener("mouseover", (e) => {
  const target = e.target.closest(".material-row .material-text");
  if (!target) return;

  let itemRaw = target.textContent.split("×")[0].trim();
  const item = itemRaw.replace(/^•\s*/, "");
  const info = tooltips[item];
  if (!info) return;

  let iconPath = "";
  if (info.icon && info.icon.trim() !== "") {
    if (info.icon.startsWith("/")) {
      iconPath = info.icon;
    } else if (info.icon.startsWith("Icon_Items.")) {
      iconPath = `/icons/Icon_Items/${info.icon.replace("Icon_Items.", "")}`;
    } else if (info.icon.startsWith("Icon_Equipment.")) {
      iconPath = `/icons/Icon_Equipment/${info.icon.replace("Icon_Equipment.", "")}`;
    } else {
      iconPath = `/icons/${info.icon}`;
    }
  }

  let iconHTML = "";
  if (iconPath) {
    iconHTML = `<img
      src="${iconPath}"
      style="width:64px;height:64px;margin-right:8px;vertical-align:top;"
      onerror="
        this.onerror=null;
        const parts=this.src.split('/');
        const file=parts.pop();

        // Split name + suffix
        const match=file.match(/^(.*)(_Tex\\.png)$/i);
        if (!match) return;

        const base=match[1];
        const suffix=match[2];

        // FIX: Uppercase first letter, lowercase the rest
        const fixed =
          base.charAt(0).toUpperCase() +
          base.slice(1).toLowerCase() +
          suffix;

        this.src = parts.join('/') + '/' + fixed;
      "
    >`;
  }

  const nameHTML = `<strong>${item}</strong><br>`;

  tooltipEl.innerHTML = `
    <div style="display:flex; align-items:flex-start;">
      ${iconHTML}
      <div style="flex:1;">${nameHTML}${formatTooltip(info.tooltip, item)}</div>
    </div>
  `;
  tooltipEl.style.display = "block";
});




document.addEventListener("mousemove", (e) => {
    if (tooltipEl.style.display === "none") return;

    const offset = 15; // distance from mouse
    let x = e.pageX + offset;
    let y = e.pageY + offset;

    // prevent tooltip from going off-screen
    const tooltipRect = tooltipEl.getBoundingClientRect();
    const winWidth = window.innerWidth;
    const winHeight = window.innerHeight;

    if (x + tooltipRect.width > winWidth - 10) x = winWidth - tooltipRect.width - 10;
    if (y + tooltipRect.height > winHeight - 10) y = winHeight - tooltipRect.height - 10;

    tooltipEl.style.left = x + "px";
    tooltipEl.style.top = y + "px";
});

document.addEventListener("mouseout", (e) => {
    if (e.target.closest(".material-text")) {
        tooltipEl.style.display = "none";
    }
});


function formatTooltip(str, itemName) {
  if (!str) return "";

  // Replace line breaks
  str = str.replace(/\$BR/g, "<br>");

  // Bold/Tag formatting
  str = str.replace(/\$B\[(.*?)\]/g, (_, text) => `<span class="tooltip-tag">${text}</span>`);

  // Good/Bad coloring
  str = str.replace(/\$H_W_GOOD(.*?)\$COLOR_END/g, (_, text) => `<span class="tooltip-good">${text}</span>`);
  str = str.replace(/\$H_W_BAD(.*?)\$COLOR_END/g, (_, text) => `<span class="tooltip-bad">${text}</span>`);

  // Remove leftover color codes
  str = str.replace(/\$COLOR_END/g, "");

  // Replace placeholders from JSON
  if (itemDurations[itemName]) {
    const data = itemDurations[itemName];
    if (data.time) str = str.replace(/\$time/g, data.time);
    if (data.duration) str = str.replace(/\$duration/g, data.duration);
    if (data.value !== undefined) str = str.replace(/\$value/g, data.value);
  } else {
    str = str.replace(/\$time/g, "").replace(/\$duration/g, "").replace(/\$value/g, "");
  }

  return str;
}



const totalsWrapper = document.getElementById("totals-wrapper");
totalsWrapper.addEventListener("click", (e) => {
  const rect = totalsWrapper.getBoundingClientRect();
  if (e.clientX < rect.right - 30) return; // only rightmost 30px triggers

  const craftedItem = document.getElementById("itemInput").value.trim();
  const qty = parseInt(document.getElementById("itemQty").value, 10) || 1;

  if (!craftedItem || !recipes[craftedItem]) {
    errorBox.textContent = "⚠ Please select a recipe before copying.";
  }

  const recipe = recipes[craftedItem];
  const buffCrit = getActiveBuffCrit();
  const buffTime = getActiveBuffTime();
  const baseTime = recipe.craftTime || 20; // base craft time
  const finalTime = Math.max(baseTime * (1 - buffTime), 1);
  const critChance = Math.min((recipe.critChance ?? recipe.crit ?? 0) + buffCrit, 1);
  const expectedCrits = Math.round(qty * critChance);

  // Copy materials from totals
  const materialsCopy = { ...lastCalculatedTotals };

  // Multiply by qty
  for (const key in materialsCopy) {
    materialsCopy[key] = Math.round(materialsCopy[key] * qty);
  }

  // Include crit yields
  const critYield = recipe.critYield || {};
  for (const [critItem, critQty] of Object.entries(critYield)) {
    const expected = critQty * expectedCrits;
    if (expected >= 1) materialsCopy[critItem] = (materialsCopy[critItem] || 0) + Math.round(expected);
  }

  // --- Compose text ---
  let text = `=================================\n`;
  text += `Item: ${craftedItem} × ${qty}\n`;
  
  // text += `Total Crit Chance: ${(critChance*100).toFixed(1)}%\n`;
  // text += `Expected Crits: ${expectedCrits}\n`;
  // text += `Craft Time per Item: ${finalTime.toFixed(2)}s\n`;
  // const totalTimeSeconds = qty * finalTime;
  // const hours = Math.floor(totalTimeSeconds / 3600);
  // const minutes = Math.floor((totalTimeSeconds % 3600) / 60);
  // const seconds = Math.floor(totalTimeSeconds % 60);
  // text += `Total Session Time: ${hours > 0 ? hours + "h " : ""}${minutes > 0 ? minutes + "m " : ""}${seconds}s\n\n`;

  text += `======== Materials Required ========\n`;
  let grandTotal = 0;

  for (const [name, amt] of Object.entries(materialsCopy)) {
    if (name.endsWith("_basePrice")) continue;

    // Water bottle items
    if (baseMaterials[name]) {
      const { baseItem, amount } = baseMaterials[name];
      const unitPrice = prices[baseItem] || 0;
      const totalBase = amt * amount;
      const totalCost = Math.floor(unitPrice * totalBase * (1 + taxPercent / 100));

      text += `• ${name} × ${formatNumberEU(amt)}\n`;
      text += `  → ${baseItem} × ${formatNumberEU(totalBase)}`;
      if (unitPrice) {
        text += ` @ ${formatMoney(unitPrice)} → ${formatMoney(totalCost)}`;
        grandTotal += totalCost;
      }
      text += `\n`;
      continue;
    }

    // Normal items
    const unitPrice = prices[name] || 0;
    const cost = unitPrice ? Math.floor(unitPrice * amt * (1 + taxPercent / 100)) : 0;

    text += `• ${name} × ${formatNumberEU(amt)}`;
    if (unitPrice) text += ` @ ${formatMoney(unitPrice)}`;
    if (cost) {
      text += ` → ${formatMoney(cost)}`;
      grandTotal += cost;
    }
    text += `\n`;
  }

  if (grandTotal > 0) {
    text += `====================\nTotal Cost: ${formatMoney(grandTotal)}\n`;
  }

  // --- Copy and toast ---
  copyToClipboard(text)
    .then(() => showToast("Crafting copied ✓"))
    .catch(err => {
      console.error("Copy failed:", err);
      alert("Failed to copy!");
    });
});





const buffContainer = document.getElementById("buff-options");
const COOKING_PROFESSION_ID = "102";
const ADDITIVE_ALLOWED_PROFESSIONS = new Set(["101", "103", "104", "106"]); // allowed for general additives

function renderBuffOptions() {
  buffContainer.innerHTML = "";

  const currentItem = document.getElementById("itemInput")?.value.trim();
  const recipeKey = Object.keys(recipes).find(
    k => k.toLowerCase() === currentItem?.toLowerCase()
  );
  const recipe = recipeKey ? recipes[recipeKey] : null;

  Object.entries(craftBuffs).forEach(([key, buff]) => {
    const div = document.createElement("div");
    div.className = "buff-option";

    if (buff.options) {

      // 🔒 FILTER OPTIONS HERE
      let optionsToRender = buff.options;

      if (key === "craftAdditive") {
  optionsToRender = buff.options.filter(opt => {
    if (opt.name === "Sweet Cooking Honey") {
      // ✅ allow if no recipe selected yet
      if (!recipe) return true;

      // ✅ allow only for cooking recipes
      return String(recipe.profession) === COOKING_PROFESSION_ID;
    }
    return true;
  });
}


  const optionsHTML = buff.options.map(opt => {
  const crit = Number(opt.crit) || 0;
  const time = Number(opt.time) || 0;

  let parts = [];
  if (crit > 0) parts.push(`+${(crit * 100).toFixed(1)}% crit`);
  if (time > 0) parts.push(`-${(time * 100).toFixed(1)}% time`);

  const properNames = ["Sweet Cooking Honey"]; // don't prepend buff label
  let text;

  // --- Decide text ---
  if (opt.name.toLowerCase().includes("no") || properNames.includes(opt.name)) {
    text = opt.name + (parts.length ? ` (${parts.join(", ")})` : "");
  } else if (key === "pet") {
    // Pet options show as-is
    text = opt.name + (parts.length ? ` (${parts.join(", ")})` : "");
  } else {
    // Prepend buff label for everything else
    text = `${buff.label} ${opt.name}` + (parts.length ? ` (${parts.join(", ")})` : "");
  }

  // --- Disable logic for additives ---
  let disabled = "";
  let title = text;

  if (key === "craftAdditive" && opt.name !== "No Additive") {
    if (opt.name === "Sweet Cooking Honey") {
      if (!recipe || String(recipe.profession) !== COOKING_PROFESSION_ID) {
        disabled = "disabled";
        title = "Cooking only";
      }
    } else {
      if (!recipe || !ADDITIVE_ALLOWED_PROFESSIONS.has(String(recipe.profession))) {
        disabled = "disabled";
        title = "Restricted to certain professions";
      }
    }
  }

  return `<option value="${crit}" ${disabled} title="${title}">${text}</option>`;
}).join("");



      div.innerHTML = `
        <label for="buff-${key}">${buff.label}:</label>
        <select id="buff-${key}" onchange="updateBuffResults()">
          ${optionsHTML}
        </select>
      `;
    } else {
      // Checkbox buffs (unchanged)
      const crit = Number(buff.crit) || 0;
      const time = Number(buff.time) || 0;

      let parts = [];
      if (crit > 0) parts.push(`+${(crit * 100).toFixed(1)}% crit`);
      if (time > 0) parts.push(`-${(time * 100).toFixed(1)}% time`);

      div.innerHTML = `
  <input type="checkbox"
         id="buff-${key}"
         onchange="
           handleExclusiveBuffs('${key}');
           updateBuffResults();
         ">
  <label for="buff-${key}">
    ${buff.label}${parts.length ? ` (${parts.join(", ")})` : ""}
  </label>
  <span class="buff-warning" id="buff-warning-${key}" style="display:none;">
    <span class="icon">⚠</span>
    <span class="text"></span>
  </span>
`;
    }

    buffContainer.appendChild(div);
  });
}


function getActiveBuffCrit() {
  let bonus = 0;
  for (const key in craftBuffs) {
    const buff = craftBuffs[key];
    if (buff.options) {
      const select = document.getElementById(`buff-${key}`);
      if (select) {
        const crit = Number(buff.options[select.selectedIndex]?.crit) || 0;
        bonus += crit;
      }
    } else if (buff.crit) {
      const checkbox = document.getElementById(`buff-${key}`);
      if (checkbox?.checked) bonus += buff.crit;
    }
  }
  return bonus;
}

function getActiveBuffTime() {
  let reduction = 0;
  for (const key in craftBuffs) {
    const buff = craftBuffs[key];
    if (buff.options) {
      const select = document.getElementById(`buff-${key}`);
      if (select) {
        const time = Number(buff.options[select.selectedIndex]?.time) || 0;
        reduction += time;
      }
    } else if (buff.time) {
      const checkbox = document.getElementById(`buff-${key}`);
      if (checkbox?.checked) reduction += buff.time;
    }
  }
  return Math.min(reduction, 1);
}

function updateBuffResults() {
  const rawItem = document.getElementById("itemInput").value.trim();
  const item = rawItem.replace(/^•\s*/, "").trim();
  const qty = parseInt(document.getElementById("itemQty").value, 10) || 1;
  const resultBox = document.getElementById("buff-results");

  // Case-insensitive recipe lookup
  const recipeKey = Object.keys(recipes).find(key => key.toLowerCase() === item.toLowerCase());
  if (!recipeKey) {
    resultBox.innerHTML = "<p>Item not found.</p>";
    return;
  }

  const recipe = recipes[recipeKey];

  // --- Buffs ---
  const buffCrit = getActiveBuffCrit();
  const buffTime = getActiveBuffTime();

  // --- Craft time ---
  const baseTime = recipe.craftTime || 20; // base craft time
  let finalTime = baseTime * (1 - buffTime);
  if (finalTime < 1) finalTime = 1;

  // Total session time
const totalCrafts = qty; // already from itemQty input
const totalTimeSeconds = totalCrafts * finalTime;

// Convert to minutes/hours for easier reading
const hours = Math.floor(totalTimeSeconds / 3600);
const minutes = Math.floor((totalTimeSeconds % 3600) / 60);
const seconds = Math.floor(totalTimeSeconds % 60);


  // --- Crit calculations ---
  const critChance = Math.min((recipe.critChance ?? recipe.crit ?? 0) + buffCrit, 1);
  let expectedCrits = qty * critChance;
  expectedCrits = Math.round(expectedCrits); // round expected crits to whole numbers

  // --- Expected items ---
  const expectedItems = {};
  const baseYield = recipe.craftYield ?? recipe.amount ?? 1;

  // Base yield
  expectedItems[recipeKey] = baseYield * qty;

  // Crit yield (can be multiple items)
  const critYield = recipe.critYield || {};
  for (const [critItem, critQty] of Object.entries(critYield)) {
    let expected = critQty * expectedCrits; // use rounded crits
    if (expected >= 1) {                     // hide tiny fractional crits
      expectedItems[critItem] = (expectedItems[critItem] || 0) + Math.round(expected);
    }
  }

  // --- Calculate expected extra items (for single-item crit fallback) ---
  let expectedBonus = 0;
  if (Object.keys(critYield).length === 0) {
    // if critYield not defined, assume +1 per crit
    expectedBonus = expectedCrits;
  }

  // --- Render results ---
  resultBox.innerHTML = `
    <p><strong>Buff Bonus:</strong> <span class="buff-highlight">+${(buffCrit*100).toFixed(1)}%</span></p>
    <p><strong>Total Crit:</strong> <span class="buff-highlight">+${(critChance*100).toFixed(1)}%</span></p>
    <hr>
    <p><strong>Time Reduction:</strong> <span class="buff-highlight">-${(buffTime*100).toFixed(1)}%</span></p>
    <p><strong>Time Per Craft:</strong> <span class="buff-highlight">${finalTime.toFixed(2)}s</span></p>
    <p><strong>Total Craft Time for ${totalCrafts} crafts:</strong> ${hours > 0 ? hours + "h " : ""}${minutes > 0 ? minutes + "m " : ""}${seconds}s</p>
    <hr>
    <p><strong>Expected Crits:</strong> <span class="buff-highlight">${expectedCrits}</span></p>
    ${expectedBonus > 0 ? `<p><strong>Expected Extra Items:</strong> <span class="buff-highlight">${expectedBonus}</span></p>` : ""}
    <p><strong>Expected Items:</strong></p>
    <ul>
      ${Object.entries(expectedItems)
        .map(([name, amt]) => `<li>${name} × ${Math.round(amt)}</li>`) // round all yields
        .join("")}
    </ul>
  `;
}
// resultBox.innerHTML = `
//    <p>Base Crit: <span class="buff-highlight">${(baseCrit * 100).toFixed(1)}%</span></p>
//    <p>Buff Bonus: <span class="buff-highlight">+${(buffCrit * 100).toFixed(1)}%</span></p>
//    <p>Total Crit: <span class="buff-highlight">${(totalCrit * 100).toFixed(1)}%</span></p>
//    <hr>
//    <p>Crafts: ${qty}</p>
//    <p>Expected Crits: <span class="buff-highlight">${expectedCrits.toFixed(2)}</span></p>
//    <p>Expected Extra Items: <span class="buff-highlight">${expectedBonus.toFixed(2)}</span></p>
//  `;

const bottomBar = document.getElementById("bottom-bar");
const observer = new IntersectionObserver(
  ([entry]) => {
    if (!entry.isIntersecting) {
      // bottom bar is stuck at bottom
      bottomBar.classList.add("stuck");
    } else {
      // bottom bar is not sticking yet
      bottomBar.classList.remove("stuck");
    }
  },
  {
    threshold: [1],  // fully visible triggers "not stuck"
    root: null,
  }
);

// --- Clipboard helper ---
function copyToClipboard(text) {
  if (navigator.clipboard && window.isSecureContext) {
    return navigator.clipboard.writeText(text);
  }

  // Fallback for insecure context
  return new Promise((resolve, reject) => {
    const textarea = document.createElement("textarea");
    textarea.value = text;
    textarea.style.position = "fixed";
    textarea.style.top = "-9999px";
    textarea.style.left = "-9999px";
    document.body.appendChild(textarea);
    textarea.focus();
    textarea.select();

    try {
      document.execCommand("copy");
      resolve();
    } catch (err) {
      reject(err);
    } finally {
      document.body.removeChild(textarea);
    }
  });
}

// --- Toast popup ---
function showToast(message, duration = 2500) {
  let toast = document.getElementById("copy-toast");
  if (!toast) {
    toast = document.createElement("div");
    toast.id = "copy-toast";
    toast.style.position = "fixed";
    toast.style.bottom = "80px";           // higher so it's easier to see
    toast.style.right = "50%";             // center horizontally
    toast.style.transform = "translateX(50%) translateY(20px)"; // start slightly lower
    toast.style.padding = "16px 28px";     // bigger
    toast.style.background = "rgba(50,50,50,0.95)";
    toast.style.color = "#fff";
    toast.style.fontSize = "16px";         // larger font
    toast.style.fontWeight = "600";        // more noticeable
    toast.style.borderRadius = "12px";     // rounded
    toast.style.boxShadow = "0 6px 18px rgba(0,0,0,0.4)";
    toast.style.zIndex = 9999;
    toast.style.opacity = 0;
    toast.style.transition = "opacity 0.4s, transform 0.4s";
    toast.style.pointerEvents = "none";    // ignore clicks
    document.body.appendChild(toast);
  }

  toast.textContent = message;
  toast.style.opacity = 1;
  toast.style.transform = "translateX(50%) translateY(0)"; // slide up

  setTimeout(() => {
    toast.style.opacity = 0;
    toast.style.transform = "translateX(50%) translateY(20px)"; // slide down
  }, duration);
}


// Observe a dummy sentinel above the bottom bar
const sentinel = document.createElement("div");
sentinel.style.height = "1px";
bottomBar.parentNode.insertBefore(sentinel, bottomBar);
observer.observe(sentinel);

function hideSuggestions() {
  const suggestionsDiv = document.getElementById("suggestions");
  if (suggestionsDiv) suggestionsDiv.style.display = "none";
}

document.addEventListener("click", (e) => {
  const suggestionsDiv = document.getElementById("suggestions");
  const input = document.getElementById("itemInput");
  if (!suggestionsDiv || !input) return;

  if (!suggestionsDiv.contains(e.target) && e.target !== input) {
    suggestionsDiv.style.display = "none";
  }
});

const itemInput = document.getElementById("itemInput");
const suggestionsDiv = document.getElementById("suggestions");
const errorBox = document.getElementById("error-message"); // ✅ the inline error container

// --- Enter key handler ---
// --- highlight a suggestion ---
function setHighlight(index) {
  const items = suggestionsDiv.querySelectorAll("div");
  items.forEach((div, i) => {
    if (i === index) {
      div.style.background = "#f0f0f0";  // subtle light gray instead of white
      div.style.color = "#111";           // make text darker
    } else {
      div.style.background = "";
      div.style.color = "";                // reset to default
    }
  });
  selectedIndex = index;
}


// --- Enter / Arrow navigation ---
itemInput.addEventListener("keydown", (e) => {
  const items = suggestionsDiv.querySelectorAll("div");

  if (e.key === "ArrowDown") {
    e.preventDefault();
    if (items.length === 0) return;
    let nextIndex = selectedIndex + 1;
    if (nextIndex >= items.length) nextIndex = 0;
    setHighlight(nextIndex);
  } else if (e.key === "ArrowUp") {
    e.preventDefault();
    if (items.length === 0) return;
    let prevIndex = selectedIndex - 1;
    if (prevIndex < 0) prevIndex = items.length - 1;
    setHighlight(prevIndex);
  } else if (e.key === "Enter") {
    e.preventDefault();

    const typedValue = itemInput.value.trim();

    if (items.length > 0 && selectedIndex < 0) {
      // Auto-select first suggestion if none highlighted
      selectedIndex = 0;
      setHighlight(0);
    }

    if (selectedIndex >= 0 && selectedIndex < items.length) {
      // pick highlighted or first suggestion
      selectSuggestion(items[selectedIndex].textContent.trim());
    } else {
      // no suggestions → validate typed input
      if (!typedValue || !recipes[typedValue]) {
        // ❌ show error message
        errorBox.textContent = "⚠ Please select a valid item.";
        errorBox.style.display = "block";

        // clear previous results
        document.getElementById("results").innerHTML = "";
        document.getElementById("totals-list").innerHTML = "";
        document.getElementById("buff-results").innerHTML = "<p>Select an item to see crit info.</p>";
      } else {
        // valid typed input → clear error and calculate
        errorBox.textContent = "";
        errorBox.style.display = "none";
        calculate();
      }
    }
  }
});

const suffixPriority = {
  "": 0,          // base item
  "Special": 1,   // special items
  "Marvelous": 2  // marvelous items
};

function handleExclusiveBuffs(changedKey) {
  const serverBox = document.getElementById("buff-Server");
  const gmBox = document.getElementById("buff-GM");

  const serverRow = serverBox?.closest(".buff-option");
  const gmRow = gmBox?.closest(".buff-option");

  const serverWarn = document.getElementById("buff-warning-Server");
  const gmWarn = document.getElementById("buff-warning-GM");

  if (!serverBox || !gmBox) return;

  // --- GM BUFF HAS PRIORITY ---
  if (gmBox.checked) {
    // Force-disable Server Buff
    serverBox.checked = false;
    serverBox.disabled = true;

    serverRow?.classList.add("disabled");
    if (serverWarn) {
      serverWarn.style.display = "inline-flex";
      serverWarn.querySelector(".text").textContent =
        "";
    }

    // GM remains fully active
    gmBox.disabled = false;
    gmRow?.classList.remove("disabled");
    if (gmWarn) gmWarn.style.display = "none";

    return;
  }

  // --- GM NOT ACTIVE → Server Buff allowed ---
  serverBox.disabled = false;
  serverRow?.classList.remove("disabled");
  if (serverWarn) serverWarn.style.display = "none";
}




</script>