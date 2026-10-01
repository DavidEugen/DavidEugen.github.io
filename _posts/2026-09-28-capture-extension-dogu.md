---
layout: post
title:  "capototototo화면 캡처 도구"
date:   2026-09-28 11:08:53 +0900
categories: think

---

```plain
[manifest.json]=========
{
  "manifest_version": 3,
  "name": "JSP 화면 캡처 도구",
  "description": "JSP 업무 화면의 메뉴 경로를 추적해 PNG로 저장하고 클립보드에 복사합니다.",
  "version": "1.3.1",
  "minimum_chrome_version": "109",
  "permissions": [
    "activeTab",
    "clipboardWrite",
    "downloads",
    "storage",
    "tabs"
  ],
  "host_permissions": [
    "<all_urls>"
  ],
  "background": {
    "service_worker": "service-worker.js"
  },
  "action": {
    "default_title": "JSP 화면 캡처 도구 열기",
    "default_popup": "popup.html"
  },
  "content_scripts": [
    {
      "matches": [
        "http://*/*",
        "https://*/*"
      ],
      "js": [
        "content.js"
      ],
      "all_frames": true,
      "match_about_blank": true,
      "run_at": "document_idle"
    }
  ],
  "commands": {
    "_execute_action": {
      "suggested_key": {
        "default": "Ctrl+Shift+8"
      }
    },
    "capture-visible": {
      "suggested_key": {
        "default": "Ctrl+Shift+7"
      },
      "description": "현재 탭의 보이는 화면 캡처"
    }
  }
}

[service-worker.js]=========
"use strict";

const STATE_KEY = "jspCaptureToolState";

function compactText(value) {
  return String(value ?? "").replace(/\s+/g, " ").trim();
}

function sanitizeFilenamePart(value, fallback) {
  const cleaned = compactText(value)
    .replace(/[<>:"/\\|?*\u0000-\u001F]/g, "-")
    .replace(/_+/g, "-")
    .replace(/[. ]+$/g, "")
    .slice(0, 80);
  return cleaned || fallback;
}

function sanitizeFolder(value) {
  const parts = String(value ?? "")
    .replace(/\\/g, "/")
    .split("/")
    .map((part) => sanitizeFilenamePart(part, ""))
    .filter((part) => part && part !== "." && part !== "..");
  return parts.join("/") || "화면캡처";
}

function buildDownloadFilename(values) {
  const folder = sanitizeFolder(values.folder);
  const parts = [
    sanitizeFilenamePart(values.gnb, "GNB"),
    sanitizeFilenamePart(values.subGnb, "SubGNB"),
    sanitizeFilenamePart(values.lnb, "LNB"),
    sanitizeFilenamePart(values.subLnb, "SubLNB"),
    sanitizeFilenamePart(values.action, "액션")
  ];
  return `${folder}/${parts.join("_")}.png`;
}

async function getActiveTab() {
  const tabs = await chrome.tabs.query({ active: true, currentWindow: true });
  const tab = tabs[0];
  if (!tab?.id || !/^https?:/i.test(tab.url || "")) {
    throw new Error("캡처할 웹 페이지 탭을 먼저 선택해 주세요.");
  }
  return tab;
}

async function captureVisible(values) {
  const tab = await getActiveTab();
  const dataUrl = await chrome.tabs.captureVisibleTab(tab.windowId, {
    format: "png"
  });
  const filename = buildDownloadFilename(values);
  const downloadId = await chrome.downloads.download({
    url: dataUrl,
    filename,
    conflictAction: "uniquify",
    saveAs: false
  });
  return { downloadId, filename };
}

async function valuesForShortcut(activeTab) {
  const stored = await chrome.storage.local.get(STATE_KEY);
  const state = stored[STATE_KEY] || {};
  const tab = activeTab || await getActiveTab();
  let siteKey = "default";
  try {
    siteKey = new URL(tab.url).origin;
  } catch (error) {
    // 기본 프로필을 사용합니다.
  }
  const profileValues = state.profiles?.[siteKey]?.values || {};
  const cacheKey = `menuState:${tab.id}`;
  const cached = await chrome.storage.session.get(cacheKey);
  const menuValues = {
    ...profileValues,
    ...(cached[cacheKey]?.values || {})
  };
  const values = {
    folder: state.folder || "화면캡처",
    gnb: menuValues.gnb || "",
    subGnb: menuValues.subGnb || "",
    lnb: menuValues.lnb || "",
    subLnb: menuValues.subLnb || "",
    action: state.action || "조회"
  };
  if ([values.gnb, values.subGnb, values.lnb, values.subLnb].some((value) => !compactText(value))) {
    throw new Error("확장 팝업에서 메뉴 4칸을 먼저 입력해 주세요.");
  }
  return values;
}

async function openShortcutCaptureWindow() {
  const tab = await getActiveTab();
  const values = await valuesForShortcut(tab);
  const runnerUrl = new URL(chrome.runtime.getURL("shortcut-capture.html"));
  runnerUrl.searchParams.set("windowId", String(tab.windowId));
  runnerUrl.searchParams.set("filename", buildDownloadFilename(values));

  await chrome.windows.create({
    url: runnerUrl.href,
    type: "popup",
    width: 390,
    height: 230,
    focused: true
  });
}

chrome.commands.onCommand.addListener(async (command) => {
  if (command !== "capture-visible") {
    return;
  }
  try {
    await openShortcutCaptureWindow();
  } catch (error) {
    console.warn("단축키 캡처 실패:", error);
  }
});

async function cacheMenuState(tabId, message) {
  const key = `menuState:${tabId}`;
  const stored = await chrome.storage.session.get(key);
  const previous = stored[key] || { values: {} };
  await chrome.storage.session.set({
    [key]: {
      values: { ...previous.values, ...(message.values || {}) },
      pageTitle: message.pageTitle || previous.pageTitle || "",
      updatedAt: Date.now()
    }
  });
}

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message?.type === "MENU_STATE" && sender.tab?.id) {
    cacheMenuState(sender.tab.id, message).catch(console.warn);
    return false;
  }

  if (message?.type === "GET_TAB_MENU_STATE") {
    const key = `menuState:${message.tabId}`;
    chrome.storage.session.get(key)
      .then((stored) => sendResponse({ ok: true, data: stored[key] || { values: {} } }))
      .catch((error) => sendResponse({ ok: false, error: error?.message || "메뉴 상태를 불러오지 못했습니다." }));
    return true;
  }

  if (message?.type === "CAPTURE_VISIBLE") {
    captureVisible(message.values || {})
      .then((data) => sendResponse({ ok: true, data }))
      .catch((error) => sendResponse({
        ok: false,
        error: error?.message || "화면 캡처에 실패했습니다."
      }));
    return true;
  }

  return false;
});

[content.js]=========
(function initializeJspCaptureObserver() {
  "use strict";

  if (globalThis.__JSP_CAPTURE_OBSERVER_LOADED__) {
    return;
  }
  globalThis.__JSP_CAPTURE_OBSERVER_LOADED__ = true;

  const ACTIVE_SELECTOR = [
    "[aria-current]:not([aria-current='false'])",
    "[aria-selected='true']",
    ".active",
    ".current",
    ".selected",
    ".on",
    ".is-active",
    ".is-current"
  ].join(",");

  const LEVELS = [
    {
      key: "gnb",
      activeSelectors: [
        "#mainMenu > ul.menu_dep1 > li.active",
        "#mainMenu ul.menu_dep1 > li.active"
      ],
      clickSelectors: [
        "#mainMenu > ul.menu_dep1 > li > a",
        "#mainMenu ul.menu_dep1 > li > a",
        "#mainMenu > ul.menu_dep1 > li",
        "#mainMenu ul.menu_dep1 > li"
      ],
      selectors: [
        "[data-capture-menu-level='gnb']",
        "#mainMenu",
        "#gnb",
        ".gnb",
        "[class~='gnb' i]",
        "header nav"
      ]
    },
    {
      key: "subGnb",
      activeSelectors: [
        "#mainMenu > ul.sub_menu > li.active",
        "#mainMenu ul.sub_menu > li.active"
      ],
      clickSelectors: [
        "#mainMenu > ul.sub_menu > li > a",
        "#mainMenu ul.sub_menu > li > a",
        "#mainMenu > ul.sub_menu > li",
        "#mainMenu ul.sub_menu > li"
      ],
      selectors: [
        "[data-capture-menu-level='subGnb']",
        "#mainMenu > ul.sub_menu",
        "#mainMenu ul.sub_menu",
        ".sub_menu",
        "#subGnb",
        "#sub-gnb",
        ".subGnb",
        ".sub-gnb",
        "[class*='subgnb' i]",
        "[class*='sub-gnb' i]"
      ]
    },
    {
      key: "lnb",
      activeSelectors: [
        "aside#lnb > ul#maindiv > li.depth.active",
        "#lnb ul#maindiv > li.depth.active"
      ],
      clickSelectors: [
        "aside#lnb > ul#maindiv > li.depth > strong > a",
        "#lnb ul#maindiv > li.depth > strong > a",
        "aside#lnb > ul#maindiv > li.depth > a",
        "#lnb ul#maindiv > li.depth > a"
      ],
      selectors: [
        "[data-capture-menu-level='lnb']",
        "aside#lnb",
        "#lnb",
        ".lnb",
        "[class~='lnb' i]",
        "aside nav"
      ]
    },
    {
      key: "subLnb",
      activeSelectors: [
        "aside#lnb > ul#maindiv > li.depth.active > ul > li.active",
        "#lnb ul#maindiv > li.depth.active > ul > li.active"
      ],
      clickSelectors: [
        "aside#lnb > ul#maindiv > li.depth > ul > li > a",
        "#lnb ul#maindiv > li.depth > ul > li > a",
        "aside#lnb > ul#maindiv > li.depth > ul > li",
        "#lnb ul#maindiv > li.depth > ul > li"
      ],
      selectors: [
        "[data-capture-menu-level='subLnb']",
        "aside#lnb > ul#maindiv > li.depth.active > ul",
        "#lnb ul#maindiv > li.depth.active > ul",
        "#subLnb",
        "#sub-lnb",
        ".subLnb",
        ".sub-lnb",
        "[class*='sublnb' i]",
        "[class*='sub-lnb' i]"
      ]
    }
  ];

  const runtimeValues = {
    gnb: "",
    subGnb: "",
    lnb: "",
    subLnb: ""
  };
  let scanTimer = 0;

  function compactText(value, maximum = 100) {
    const text = String(value || "").replace(/\s+/g, " ").trim();
    return text.length > maximum ? text.slice(0, maximum) : text;
  }

  function isVisible(element) {
    if (!element || element.hidden || element.getAttribute("aria-hidden") === "true") {
      return false;
    }
    const style = getComputedStyle(element);
    if (style.display === "none" || style.visibility === "hidden" || Number(style.opacity) === 0) {
      return false;
    }
    const rect = element.getBoundingClientRect();
    return rect.width > 0 && rect.height > 0;
  }

  function labelFor(element) {
    if (!element) {
      return "";
    }
    const explicit = element.getAttribute("data-capture-label")
      || element.getAttribute("aria-label")
      || element.getAttribute("title");
    if (compactText(explicit)) {
      return compactText(explicit);
    }

    const directLabel = element.matches("a,button,[role='menuitem']")
      ? element
      : element.querySelector(":scope > a, :scope > button, :scope > [role='menuitem'], :scope > span");
    const source = directLabel || element;
    const clone = source.cloneNode(true);
    clone.querySelectorAll("ul,ol,nav,[role='menu'],script,style").forEach((child) => child.remove());
    return compactText(clone.textContent);
  }

  function firstVisible(selector, root = document) {
    try {
      return Array.from(root.querySelectorAll(selector)).find(isVisible) || null;
    } catch (error) {
      return null;
    }
  }

  function findContainer(level) {
    for (const selector of level.selectors) {
      const match = firstVisible(selector);
      if (match) {
        return match;
      }
    }
    return null;
  }

  function activeLabel(container) {
    if (!container) {
      return "";
    }
    let candidates = [];
    try {
      candidates = Array.from(container.querySelectorAll(ACTIVE_SELECTOR)).filter(isVisible);
    } catch (error) {
      return "";
    }
    for (let index = candidates.length - 1; index >= 0; index -= 1) {
      const text = labelFor(candidates[index]);
      if (text) {
        return text;
      }
    }
    if (container.matches(ACTIVE_SELECTOR)) {
      return labelFor(container);
    }
    return "";
  }

  function activeLabelForLevel(level) {
    for (const selector of level.activeSelectors || []) {
      const activeItem = firstVisible(selector);
      const text = labelFor(activeItem);
      if (text) {
        return text;
      }
    }
    return activeLabel(findContainer(level));
  }

  function detectValues() {
    const detected = {};
    for (const level of LEVELS) {
      const value = activeLabelForLevel(level);
      if (value) {
        detected[level.key] = value;
      }
    }
    return detected;
  }

  function emit(values, source) {
    if (!Object.keys(values).length) {
      return;
    }
    chrome.runtime.sendMessage({
      type: "MENU_STATE",
      values,
      source,
      isTopFrame: window === window.top,
      pageTitle: document.title
    }).catch(() => undefined);
  }

  function scan(source = "mutation") {
    const detected = detectValues();
    const changed = {};
    for (const level of LEVELS) {
      const value = detected[level.key];
      if (value && value !== runtimeValues[level.key]) {
        runtimeValues[level.key] = value;
        changed[level.key] = value;
      }
    }
    emit(changed, source);
    return { ...runtimeValues };
  }

  function levelForElement(element) {
    const reversed = [...LEVELS].reverse();
    for (const level of reversed) {
      for (const selector of level.clickSelectors || []) {
        try {
          if (element.matches(selector) || element.closest(selector)) {
            return level.key;
          }
        } catch (error) {
          // 유효하지 않은 사이트 선택자는 건너뜁니다.
        }
      }
    }
    for (const level of reversed) {
      for (const selector of level.selectors) {
        try {
          if (element.matches(selector) || element.closest(selector)) {
            return level.key;
          }
        } catch (error) {
          // 유효하지 않은 사이트 선택자는 건너뜁니다.
        }
      }
    }
    return "";
  }

  document.addEventListener("click", (event) => {
    const path = typeof event.composedPath === "function" ? event.composedPath() : [event.target];
    const clicked = path.find((item) => item instanceof Element);
    if (!clicked) {
      return;
    }
    const interactive = clicked.closest("a,button,li,[role='menuitem']") || clicked;
    const level = levelForElement(interactive);
    const value = labelFor(interactive);
    if (level && value) {
      runtimeValues[level] = value;
      emit({ [level]: value }, "click");
    }
    window.clearTimeout(scanTimer);
    scanTimer = window.setTimeout(() => scan("after-click"), 250);
  }, true);

  chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
    if (message?.type !== "DETECT_MENUS") {
      return false;
    }
    sendResponse({
      ok: true,
      values: scan("manual"),
      pageTitle: document.title
    });
    return false;
  });

  const observer = new MutationObserver(() => {
    window.clearTimeout(scanTimer);
    scanTimer = window.setTimeout(() => scan("mutation"), 180);
  });

  observer.observe(document.documentElement, {
    subtree: true,
    childList: true,
    attributes: true,
    attributeFilter: ["class", "aria-current", "aria-selected", "hidden", "style"]
  });

  scan("initial");
  window.setInterval(() => scan("interval"), 1500);
})();

[popup.html]=========
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>JSP 화면 캡처</title>
    <link rel="stylesheet" href="popup.css">
  </head>
  <body>
    <header class="app-header">
      <div>
        <p class="eyebrow">EDGE EXTENSION</p>
        <h1>JSP 화면 캡처</h1>
      </div>
      <span id="connectionBadge" class="badge">확인 중</span>
    </header>

    <main>
      <form id="captureForm" novalidate>
        <section class="card" aria-labelledby="pathTitle">
          <div class="section-title">
            <span class="step-number">1</span>
            <div>
              <h2 id="pathTitle">저장 경로</h2>
              <p>Edge의 다운로드 폴더 아래 경로입니다.</p>
            </div>
          </div>

          <label class="field">
            <span>저장 경로</span>
            <input
              id="folderInput"
              name="folder"
              type="text"
              maxlength="180"
              placeholder="예: 화면캡처/회원관리"
              spellcheck="false"
              required
            >
          </label>
        </section>

        <section class="card" aria-labelledby="menuTitle">
          <div class="section-title">
            <span class="step-number">2</span>
            <div>
              <h2 id="menuTitle">메뉴 경로</h2>
              <p>감지된 메뉴를 확인하거나 직접 입력하세요.</p>
            </div>
          </div>

          <div class="menu-grid">
            <label class="field">
              <span>GNB</span>
              <input id="gnbInput" name="gnb" type="text" maxlength="100" list="gnbHistory" placeholder="대메뉴" autocomplete="off" required>
              <datalist id="gnbHistory"></datalist>
            </label>

            <label class="field">
              <span>SubGNB</span>
              <input id="subGnbInput" name="subGnb" type="text" maxlength="100" list="subGnbHistory" placeholder="중메뉴" autocomplete="off" required>
              <datalist id="subGnbHistory"></datalist>
            </label>

            <label class="field">
              <span>LNB</span>
              <input id="lnbInput" name="lnb" type="text" maxlength="100" list="lnbHistory" placeholder="좌측 메뉴" autocomplete="off" required>
              <datalist id="lnbHistory"></datalist>
            </label>

            <label class="field">
              <span>SubLNB</span>
              <input id="subLnbInput" name="subLnb" type="text" maxlength="100" list="subLnbHistory" placeholder="좌측 하위 메뉴" autocomplete="off" required>
              <datalist id="subLnbHistory"></datalist>
            </label>
          </div>

          <div class="detect-row">
            <span id="detectMessage">현재 화면의 메뉴 상태를 확인합니다.</span>
            <button id="detectButton" class="text-button" type="button">다시 감지</button>
          </div>
        </section>

        <section class="card" aria-labelledby="actionTitle">
          <div class="section-title">
            <span class="step-number">3</span>
            <div>
              <h2 id="actionTitle">액션</h2>
              <p>화면 상태를 설명하는 마지막 이름입니다.</p>
            </div>
          </div>

          <label class="field">
            <span>액션명</span>
            <input id="actionInput" name="action" type="text" maxlength="100" list="actionSuggestions" placeholder="예: 조회, 저장, 수정, 삭제" autocomplete="off" required>
            <datalist id="actionSuggestions">
              <option value="조회"></option>
              <option value="등록"></option>
              <option value="저장"></option>
              <option value="수정"></option>
              <option value="삭제"></option>
              <option value="검색"></option>
              <option value="상세"></option>
              <option value="다운로드"></option>
              <option value="인쇄"></option>
            </datalist>
          </label>
        </section>

        <section class="preview-card" aria-live="polite">
          <span>저장될 파일</span>
          <strong id="filenamePreview">입력값을 작성해 주세요.</strong>
        </section>

        <button id="captureButton" class="capture-button" type="submit">
          <span class="camera-icon" aria-hidden="true"></span>
          <span>
            <strong>현재 화면 캡처</strong>
            <small>PNG로 저장하고 클립보드에도 복사</small>
          </span>
        </button>

        <p id="statusMessage" class="status-message" role="status"></p>
        <button id="copyAgainButton" class="retry-copy-button" type="button" hidden>
          클립보드 다시 복사
        </button>
      </form>
    </main>

    <script src="popup.js"></script>
  </body>
</html>

[popup.css]=========
:root {
  color-scheme: light;
  font-family: "Pretendard", "Noto Sans KR", "Malgun Gothic", system-ui, sans-serif;
  color: #172033;
  background: #f3f6fb;
  font-synthesis: none;
}

* {
  box-sizing: border-box;
}

html,
body {
  width: 400px;
}

body {
  margin: 0;
  min-width: 400px;
  min-height: 100vh;
  background:
    radial-gradient(circle at 100% 0, rgba(55, 107, 246, 0.11), transparent 240px),
    #f3f6fb;
}

button,
input {
  font: inherit;
}

.app-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding: 22px 20px 17px;
  border-bottom: 1px solid #dfe5ef;
  background: rgba(255, 255, 255, 0.82);
  backdrop-filter: blur(12px);
  position: sticky;
  top: 0;
  z-index: 10;
}

.eyebrow {
  margin: 0 0 3px;
  color: #5f6d86;
  font-size: 10px;
  font-weight: 800;
  letter-spacing: 0.12em;
}

h1 {
  margin: 0;
  color: #111a2c;
  font-size: 21px;
  letter-spacing: -0.04em;
}

.badge {
  flex: none;
  padding: 6px 9px;
  border: 1px solid #d7deea;
  border-radius: 999px;
  color: #68748a;
  background: #f7f9fc;
  font-size: 11px;
  font-weight: 700;
}

.badge.connected {
  border-color: #b7e4d1;
  color: #087b54;
  background: #eaf9f2;
}

main {
  padding: 14px;
}

.card {
  margin-bottom: 10px;
  padding: 16px;
  border: 1px solid #e0e6ef;
  border-radius: 14px;
  background: #fff;
  box-shadow: 0 4px 16px rgba(39, 54, 84, 0.045);
}

.section-title {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  margin-bottom: 15px;
}

.step-number {
  display: grid;
  place-items: center;
  width: 25px;
  height: 25px;
  flex: none;
  border-radius: 8px;
  color: #fff;
  background: #376bf6;
  font-size: 12px;
  font-weight: 800;
  box-shadow: 0 4px 10px rgba(55, 107, 246, 0.25);
}

.section-title h2 {
  margin: 1px 0 3px;
  color: #1b2538;
  font-size: 14px;
  letter-spacing: -0.025em;
}

.section-title p {
  margin: 0;
  color: #7a869a;
  font-size: 11px;
  line-height: 1.45;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.field > span {
  color: #4b5870;
  font-size: 11px;
  font-weight: 750;
}

.field input {
  width: 100%;
  height: 39px;
  padding: 0 11px;
  border: 1px solid #d7deea;
  border-radius: 9px;
  outline: none;
  color: #182238;
  background: #fbfcfe;
  font-size: 13px;
  transition: border-color 120ms ease, box-shadow 120ms ease, background 120ms ease;
}

.field input:hover {
  border-color: #b9c5d8;
  background: #fff;
}

.field input:focus {
  border-color: #376bf6;
  background: #fff;
  box-shadow: 0 0 0 3px rgba(55, 107, 246, 0.12);
}

.field input:invalid:not(:placeholder-shown) {
  border-color: #db5664;
}

.menu-grid {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
  gap: 12px 10px;
}

.detect-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  margin-top: 13px;
  padding-top: 11px;
  border-top: 1px solid #edf0f5;
  color: #7a869a;
  font-size: 10px;
  line-height: 1.45;
}

.text-button {
  flex: none;
  padding: 5px 7px;
  border: 0;
  color: #315fd7;
  background: transparent;
  font-size: 11px;
  font-weight: 750;
  cursor: pointer;
}

.text-button:hover {
  text-decoration: underline;
}

.preview-card {
  margin: 13px 2px 10px;
  padding: 11px 12px;
  border: 1px dashed #c8d2e2;
  border-radius: 10px;
  background: rgba(255, 255, 255, 0.68);
}

.preview-card span {
  display: block;
  margin-bottom: 4px;
  color: #7a869a;
  font-size: 10px;
  font-weight: 700;
}

.preview-card strong {
  display: block;
  overflow-wrap: anywhere;
  color: #40506a;
  font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
  font-size: 10px;
  font-weight: 600;
  line-height: 1.45;
}

.capture-button {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  width: 100%;
  min-height: 58px;
  padding: 10px 16px;
  border: 0;
  border-radius: 13px;
  color: #fff;
  background: linear-gradient(135deg, #376bf6, #274fc3);
  box-shadow: 0 8px 20px rgba(45, 91, 218, 0.27);
  cursor: pointer;
  transition: transform 120ms ease, box-shadow 120ms ease, opacity 120ms ease;
}

.capture-button:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 10px 24px rgba(45, 91, 218, 0.34);
}

.capture-button:active:not(:disabled) {
  transform: translateY(0);
}

.capture-button:disabled {
  cursor: wait;
  opacity: 0.72;
}

.capture-button > span:last-child {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 2px;
}

.capture-button strong {
  font-size: 14px;
}

.capture-button small {
  color: rgba(255, 255, 255, 0.76);
  font-size: 10px;
}

.camera-icon {
  position: relative;
  width: 23px;
  height: 17px;
  border: 2px solid currentColor;
  border-radius: 5px;
}

.camera-icon::before {
  content: "";
  position: absolute;
  top: -6px;
  left: 5px;
  width: 8px;
  height: 5px;
  border-radius: 3px 3px 0 0;
  background: currentColor;
}

.camera-icon::after {
  content: "";
  position: absolute;
  top: 3px;
  left: 7px;
  width: 5px;
  height: 5px;
  border: 2px solid currentColor;
  border-radius: 50%;
}

.capture-button.busy .camera-icon {
  border-radius: 50%;
  border-right-color: transparent;
  animation: spin 800ms linear infinite;
}

.capture-button.busy .camera-icon::before,
.capture-button.busy .camera-icon::after {
  display: none;
}

.status-message {
  min-height: 19px;
  margin: 9px 2px 0;
  color: #6f7b90;
  font-size: 10px;
  line-height: 1.5;
  overflow-wrap: anywhere;
}

.status-message.success {
  color: #087b54;
}

.status-message.error {
  color: #c43749;
}

.status-message.warning {
  color: #a35b00;
}

.status-message.working {
  color: #315fd7;
}

.retry-copy-button {
  width: 100%;
  margin-top: 2px;
  padding: 8px 10px;
  border: 1px solid #d6a550;
  border-radius: 8px;
  color: #8a4e00;
  background: #fff8e9;
  font-size: 11px;
  font-weight: 750;
  cursor: pointer;
}

.retry-copy-button:hover {
  background: #fff2d4;
}

.retry-copy-button[hidden] {
  display: none;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

[popup.js]=========
"use strict";

// 툴바 팝업이 닫혀도 모든 입력값은 chrome.storage.local에 유지됩니다.

const STATE_KEY = "jspCaptureToolState";
const MENU_KEYS = ["gnb", "subGnb", "lnb", "subLnb"];
const HISTORY_LIMIT = 40;

const elements = {
  form: document.querySelector("#captureForm"),
  folder: document.querySelector("#folderInput"),
  gnb: document.querySelector("#gnbInput"),
  subGnb: document.querySelector("#subGnbInput"),
  lnb: document.querySelector("#lnbInput"),
  subLnb: document.querySelector("#subLnbInput"),
  action: document.querySelector("#actionInput"),
  captureButton: document.querySelector("#captureButton"),
  detectButton: document.querySelector("#detectButton"),
  filenamePreview: document.querySelector("#filenamePreview"),
  detectMessage: document.querySelector("#detectMessage"),
  statusMessage: document.querySelector("#statusMessage"),
  copyAgainButton: document.querySelector("#copyAgainButton"),
  connectionBadge: document.querySelector("#connectionBadge")
};

let state = {
  folder: "화면캡처",
  action: "조회",
  histories: { gnb: [], subGnb: [], lnb: [], subLnb: [] },
  profiles: {}
};
let currentTabId = null;
let currentWindowId = null;
let currentSiteKey = "default";
let saveTimer = 0;
let initialized = false;
let latestCaptureDataUrl = "";
let latestCaptureBlob = null;

function compactText(value) {
  return String(value ?? "").replace(/\s+/g, " ").trim();
}

function sanitizePart(value, fallback = "") {
  return compactText(value)
    .replace(/[<>:"/\\|?*\u0000-\u001F]/g, "-")
    .replace(/_+/g, "-")
    .replace(/[. ]+$/g, "")
    .slice(0, 80) || fallback;
}

function sanitizeFolder(value) {
  return String(value ?? "")
    .replace(/\\/g, "/")
    .split("/")
    .map((part) => sanitizePart(part))
    .filter((part) => part && part !== "." && part !== "..")
    .join("/") || "화면캡처";
}

function buildDownloadFilename(values) {
  const parts = [
    sanitizePart(values.gnb, "GNB"),
    sanitizePart(values.subGnb, "SubGNB"),
    sanitizePart(values.lnb, "LNB"),
    sanitizePart(values.subLnb, "SubLNB"),
    sanitizePart(values.action, "액션")
  ];
  return `${sanitizeFolder(values.folder)}/${parts.join("_")}.png`;
}

async function pngBlobFromDataUrl(dataUrl) {
  const response = await fetch(dataUrl);
  const blob = await response.blob();
  return blob.type === "image/png"
    ? blob
    : new Blob([await blob.arrayBuffer()], { type: "image/png" });
}

function writePngBlobToClipboard(pngBlobOrPromise) {
  if (!navigator.clipboard?.write || typeof ClipboardItem === "undefined") {
    return Promise.reject(new Error("이 Edge 버전에서는 이미지 클립보드를 사용할 수 없습니다."));
  }

  try {
    return navigator.clipboard.write([
      new ClipboardItem({ "image/png": pngBlobOrPromise })
    ]);
  } catch (error) {
    return Promise.reject(error);
  }
}

function copyPngToClipboard(dataUrlOrPromise) {
  const pngBlobPromise = Promise.resolve(dataUrlOrPromise).then(pngBlobFromDataUrl);
  return writePngBlobToClipboard(pngBlobPromise);
}

function normalizedState(value) {
  const saved = value && typeof value === "object" ? value : {};
  const histories = {};
  for (const key of MENU_KEYS) {
    histories[key] = Array.isArray(saved.histories?.[key])
      ? saved.histories[key].filter(Boolean).slice(0, HISTORY_LIMIT)
      : [];
  }
  return {
    folder: saved.folder || "화면캡처",
    action: saved.action || "조회",
    histories,
    profiles: saved.profiles && typeof saved.profiles === "object" ? saved.profiles : {}
  };
}

function siteKeyForUrl(url) {
  try {
    return new URL(url).origin;
  } catch (error) {
    return "default";
  }
}

function ensureProfile() {
  if (!state.profiles[currentSiteKey]) {
    state.profiles[currentSiteKey] = { values: {} };
  }
  if (!state.profiles[currentSiteKey].values) {
    state.profiles[currentSiteKey].values = {};
  }
  return state.profiles[currentSiteKey];
}

function valuesFromForm() {
  return {
    folder: compactText(elements.folder.value),
    gnb: compactText(elements.gnb.value),
    subGnb: compactText(elements.subGnb.value),
    lnb: compactText(elements.lnb.value),
    subLnb: compactText(elements.subLnb.value),
    action: compactText(elements.action.value)
  };
}

function renderPreview() {
  const values = valuesFromForm();
  const filename = [values.gnb, values.subGnb, values.lnb, values.subLnb, values.action]
    .map((value) => sanitizePart(value, "…"))
    .join("_");
  elements.filenamePreview.textContent = `다운로드/${sanitizeFolder(values.folder)}/${filename}.png`;
}

function renderHistories() {
  for (const key of MENU_KEYS) {
    const list = document.querySelector(`#${key}History`);
    list.replaceChildren();
    for (const value of state.histories[key]) {
      const option = document.createElement("option");
      option.value = value;
      list.append(option);
    }
  }
}

async function saveState() {
  state.folder = compactText(elements.folder.value) || "화면캡처";
  state.action = compactText(elements.action.value) || "조회";
  const profile = ensureProfile();
  for (const key of MENU_KEYS) {
    profile.values[key] = compactText(elements[key].value);
  }
  await chrome.storage.local.set({ [STATE_KEY]: state });
}

function scheduleSave() {
  window.clearTimeout(saveTimer);
  saveTimer = window.setTimeout(() => saveState().catch(console.error), 80);
}

function rememberMenuValues(values) {
  for (const key of MENU_KEYS) {
    const value = compactText(values[key]);
    if (!value) {
      continue;
    }
    state.histories[key] = [value, ...state.histories[key].filter((item) => item !== value)]
      .slice(0, HISTORY_LIMIT);
  }
  renderHistories();
}

function setConnection(connected) {
  elements.connectionBadge.textContent = connected ? "연결됨" : "직접 입력";
  elements.connectionBadge.classList.toggle("connected", connected);
}

function setStatus(message, type = "") {
  elements.statusMessage.textContent = message;
  elements.statusMessage.className = `status-message ${type}`.trim();
}

function applyDetected(values, source = "auto") {
  let count = 0;
  const profile = ensureProfile();
  for (const key of MENU_KEYS) {
    const value = compactText(values?.[key]);
    if (!value || document.activeElement === elements[key]) {
      continue;
    }
    if (elements[key].value !== value) {
      elements[key].value = value;
      profile.values[key] = value;
      count += 1;
    }
  }
  if (count) {
    const description = source === "click" ? "메뉴 클릭" : "화면 변경";
    elements.detectMessage.textContent = `${description}을 반영했습니다. 감지되지 않은 값은 이전 값을 유지합니다.`;
    renderPreview();
    scheduleSave();
  }
}

async function requestMenuDetection() {
  if (!currentTabId) {
    setConnection(false);
    return;
  }

  try {
    const cached = await chrome.runtime.sendMessage({
      type: "GET_TAB_MENU_STATE",
      tabId: currentTabId
    });
    if (cached?.ok) {
      applyDetected(cached.data?.values, "cache");
    }

    const response = await chrome.tabs.sendMessage(currentTabId, { type: "DETECT_MENUS" });
    if (!response?.ok) {
      throw new Error("응답 없음");
    }
    applyDetected(response.values, "manual");
    setConnection(true);
    elements.detectMessage.textContent = "현재 화면을 확인했습니다. 감지되지 않은 값은 이전 값을 유지합니다.";
  } catch (error) {
    setConnection(false);
    elements.detectMessage.textContent = "이 페이지에서는 자동 감지를 사용할 수 없어 직접 입력합니다.";
  }
}

async function loadActiveTab() {
  const tabs = await chrome.tabs.query({ active: true, currentWindow: true });
  const tab = tabs[0];
  currentTabId = tab?.id || null;
  currentWindowId = Number.isInteger(tab?.windowId) ? tab.windowId : undefined;
  currentSiteKey = siteKeyForUrl(tab?.url || "");
  const profile = ensureProfile();

  elements.folder.value = state.folder;
  elements.action.value = state.action;
  for (const key of MENU_KEYS) {
    elements[key].value = profile.values[key] || "";
  }
  renderPreview();
  await requestMenuDetection();
}

async function initialize() {
  const stored = await chrome.storage.local.get(STATE_KEY);
  state = normalizedState(stored[STATE_KEY]);
  renderHistories();
  await loadActiveTab();
  initialized = true;
}

for (const input of [elements.folder, elements.gnb, elements.subGnb, elements.lnb, elements.subLnb, elements.action]) {
  input.addEventListener("input", () => {
    renderPreview();
    scheduleSave();
  });
}

for (const key of MENU_KEYS) {
  elements[key].addEventListener("change", () => {
    rememberMenuValues({ [key]: elements[key].value });
    scheduleSave();
  });
}

elements.detectButton.addEventListener("click", () => {
  requestMenuDetection().catch(console.error);
});

elements.copyAgainButton.addEventListener("click", () => {
  if (!latestCaptureDataUrl) {
    setStatus("다시 복사할 캡처 이미지가 없습니다.", "error");
    return;
  }

  elements.copyAgainButton.disabled = true;
  // 재시도할 때는 이미 만들어 둔 Blob을 즉시 전달해 Edge 109 호환성을 높입니다.
  const copyResult = (latestCaptureBlob
    ? writePngBlobToClipboard(latestCaptureBlob)
    : copyPngToClipboard(latestCaptureDataUrl))
    .then(() => ({ ok: true }))
    .catch((error) => ({ ok: false, error }));

  copyResult.then((result) => {
    elements.copyAgainButton.disabled = false;
    if (result.ok) {
      elements.copyAgainButton.hidden = true;
      setStatus("클립보드에 PNG 이미지를 다시 복사했습니다.", "success");
    } else {
      setStatus(`클립보드 복사 실패: ${result.error?.message || "알 수 없는 오류"}`, "error");
    }
  });
});

elements.form.addEventListener("submit", async (event) => {
  event.preventDefault();
  if (!elements.form.reportValidity()) {
    setStatus("6개 입력칸을 모두 작성해 주세요.", "error");
    return;
  }

  const values = valuesFromForm();
  rememberMenuValues(values);
  elements.captureButton.disabled = true;
  elements.captureButton.classList.add("busy");
  elements.copyAgainButton.hidden = true;
  latestCaptureBlob = null;
  setStatus("현재 화면을 저장하고 클립보드에 복사하고 있습니다…", "working");

  // Edge 109에서도 사용자 동작 권한을 유지하도록 첫 await 전에 복사를 예약합니다.
  const capturePromise = chrome.tabs.captureVisibleTab(currentWindowId, { format: "png" });
  const pngBlobPromise = capturePromise.then(pngBlobFromDataUrl);
  pngBlobPromise
    .then((blob) => {
      latestCaptureBlob = blob;
    })
    .catch(() => undefined);
  const clipboardResultPromise = writePngBlobToClipboard(pngBlobPromise)
    .then(() => ({ ok: true }))
    .catch((error) => ({ ok: false, error }));

  try {
    await saveState();
    const dataUrl = await capturePromise;
    latestCaptureDataUrl = dataUrl;
    const filename = buildDownloadFilename(values);

    await chrome.downloads.download({
      url: dataUrl,
      filename,
      conflictAction: "uniquify",
      saveAs: false
    });

    const clipboardResult = await clipboardResultPromise;
    if (!clipboardResult.ok) {
      const clipboardError = clipboardResult.error?.message || "알 수 없는 오류";
      elements.copyAgainButton.hidden = false;
      setStatus(`파일은 저장했지만 클립보드 복사에 실패했습니다: ${clipboardError}`, "warning");
    } else {
      setStatus(`저장 및 클립보드 복사 완료: 다운로드/${filename}`, "success");
    }
  } catch (error) {
    setStatus(error?.message || "화면 캡처에 실패했습니다.", "error");
  } finally {
    elements.captureButton.disabled = false;
    elements.captureButton.classList.remove("busy");
  }
});

chrome.runtime.onMessage.addListener((message, sender) => {
  if (message?.type !== "MENU_STATE" || sender.tab?.id !== currentTabId) {
    return false;
  }
  applyDetected(message.values, message.source);
  setConnection(true);
  return false;
});

chrome.tabs.onActivated.addListener(() => {
  loadActiveTab().catch(console.error);
});

chrome.tabs.onUpdated.addListener((tabId, changeInfo) => {
  if (tabId === currentTabId && (changeInfo.status === "complete" || changeInfo.url)) {
    loadActiveTab().catch(console.error);
  }
});

window.addEventListener("pagehide", () => {
  if (!initialized) {
    return;
  }
  window.clearTimeout(saveTimer);
  saveState().catch(console.error);
});

initialize().catch((error) => {
  setConnection(false);
  setStatus(error?.message || "확장 프로그램을 초기화하지 못했습니다.", "error");
});

[shortcut-capture.html]=========
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>단축키 화면 캡처</title>
    <link rel="stylesheet" href="shortcut-capture.css">
  </head>
  <body>
    <main class="runner-card">
      <div id="runnerIcon" class="runner-icon" aria-hidden="true"></div>
      <div class="runner-copy">
        <h1>단축키 화면 캡처</h1>
        <p id="runnerStatus" role="status">PNG 저장과 클립보드 복사를 준비하고 있습니다…</p>
      </div>
      <div class="runner-actions">
        <button id="retryButton" type="button" hidden>클립보드 다시 복사</button>
        <button id="closeButton" type="button" hidden>닫기</button>
      </div>
    </main>
    <script src="shortcut-capture.js"></script>
  </body>
</html>

[shortcut-capture.css]=========
:root {
  color-scheme: light;
  font-family: "Pretendard", "Noto Sans KR", "Malgun Gothic", system-ui, sans-serif;
  color: #172033;
  background: #f3f6fb;
}

* {
  box-sizing: border-box;
}

body {
  display: grid;
  min-width: 360px;
  min-height: 190px;
  margin: 0;
  padding: 18px;
  place-items: center;
  background: radial-gradient(circle at 100% 0, rgba(55, 107, 246, 0.12), transparent 210px), #f3f6fb;
}

.runner-card {
  width: 100%;
  padding: 18px;
  border: 1px solid #dfe5ef;
  border-radius: 14px;
  background: #fff;
  box-shadow: 0 8px 25px rgba(39, 54, 84, 0.1);
}

.runner-icon {
  float: left;
  width: 25px;
  height: 25px;
  margin: 1px 12px 22px 0;
  border: 3px solid #376bf6;
  border-right-color: transparent;
  border-radius: 50%;
  animation: spin 750ms linear infinite;
}

.runner-icon.done {
  display: grid;
  border: 0;
  color: #fff;
  background: #087b54;
  animation: none;
  place-items: center;
}

.runner-icon.done::before {
  content: "✓";
  font-weight: 900;
}

.runner-icon.error {
  display: grid;
  border: 0;
  color: #fff;
  background: #c43749;
  animation: none;
  place-items: center;
}

.runner-icon.error::before {
  content: "!";
  font-weight: 900;
}

.runner-copy h1 {
  margin: 0 0 5px;
  font-size: 16px;
}

.runner-copy p {
  min-height: 36px;
  margin: 0;
  color: #68748a;
  font-size: 11px;
  line-height: 1.55;
  overflow-wrap: anywhere;
}

.runner-actions {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
  margin-top: 12px;
}

.runner-actions button {
  padding: 7px 11px;
  border: 1px solid #d7deea;
  border-radius: 8px;
  color: #40506a;
  background: #f7f9fc;
  font: inherit;
  font-size: 11px;
  font-weight: 750;
  cursor: pointer;
}

.runner-actions button:first-child {
  border-color: #d6a550;
  color: #8a4e00;
  background: #fff8e9;
}

[hidden] {
  display: none !important;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

[shortcut-capture.js]=========
"use strict";

const elements = {
  icon: document.querySelector("#runnerIcon"),
  status: document.querySelector("#runnerStatus"),
  retryButton: document.querySelector("#retryButton"),
  closeButton: document.querySelector("#closeButton")
};

let latestPngBlob = null;

function setResult(message, type) {
  elements.status.textContent = message;
  elements.icon.className = `runner-icon ${type}`.trim();
}

async function pngBlobFromDataUrl(dataUrl) {
  const response = await fetch(dataUrl);
  const blob = await response.blob();
  return blob.type === "image/png"
    ? blob
    : new Blob([await blob.arrayBuffer()], { type: "image/png" });
}

function writePngBlobToClipboard(pngBlobOrPromise) {
  if (!navigator.clipboard?.write || typeof ClipboardItem === "undefined") {
    return Promise.reject(new Error("이 Edge 버전에서는 이미지 클립보드를 사용할 수 없습니다."));
  }

  try {
    return navigator.clipboard.write([
      new ClipboardItem({ "image/png": pngBlobOrPromise })
    ]);
  } catch (error) {
    return Promise.reject(error);
  }
}

async function runShortcutCapture() {
  const params = new URLSearchParams(location.search);
  const sourceWindowId = Number(params.get("windowId"));
  const filename = params.get("filename") || "화면캡처/단축키_캡처.png";
  if (!Number.isInteger(sourceWindowId)) {
    throw new Error("캡처할 원본 창 정보를 찾지 못했습니다.");
  }

  // 포커스된 확장 창에서 첫 await 전에 클립보드 쓰기를 예약합니다.
  const capturePromise = chrome.tabs.captureVisibleTab(sourceWindowId, { format: "png" });
  const pngBlobPromise = capturePromise.then(pngBlobFromDataUrl);
  pngBlobPromise
    .then((blob) => {
      latestPngBlob = blob;
    })
    .catch(() => undefined);
  const clipboardResultPromise = writePngBlobToClipboard(pngBlobPromise)
    .then(() => ({ ok: true }))
    .catch((error) => ({ ok: false, error }));

  const dataUrl = await capturePromise;
  await chrome.downloads.download({
    url: dataUrl,
    filename,
    conflictAction: "uniquify",
    saveAs: false
  });

  const clipboardResult = await clipboardResultPromise;
  if (!clipboardResult.ok) {
    elements.retryButton.hidden = false;
    elements.closeButton.hidden = false;
    setResult(
      `파일은 저장했지만 클립보드 복사에 실패했습니다: ${clipboardResult.error?.message || "알 수 없는 오류"}`,
      "error"
    );
    return;
  }

  setResult("PNG 저장과 클립보드 복사를 완료했습니다.", "done");
  window.setTimeout(() => window.close(), 900);
}

elements.retryButton.addEventListener("click", () => {
  if (!latestPngBlob) {
    setResult("다시 복사할 PNG 데이터가 없습니다.", "error");
    return;
  }

  elements.retryButton.disabled = true;
  writePngBlobToClipboard(latestPngBlob)
    .then(() => {
      setResult("클립보드에 PNG 이미지를 다시 복사했습니다.", "done");
      window.setTimeout(() => window.close(), 900);
    })
    .catch((error) => {
      elements.retryButton.disabled = false;
      setResult(`클립보드 복사 실패: ${error?.message || "알 수 없는 오류"}`, "error");
    });
});

elements.closeButton.addEventListener("click", () => window.close());

runShortcutCapture().catch((error) => {
  elements.closeButton.hidden = false;
  setResult(error?.message || "단축키 캡처에 실패했습니다.", "error");
});



```

