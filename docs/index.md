---
title: Converter
hide:
  - navigation
  - toc
---

<style>
    .container {
        display: flex;
        gap: 30px;
    }
    @media (max-width: 800px) {
        .container {
            flex-direction: column;
        }
    }
    .container > div {
        flex: 1;
        height: 100%;
    }
    #markdown, #html {
        border: 1px solid var(--md-typeset-table-color);
        border-radius: 5px;
        height: 55vh;
        width: 100%;
        font-size: .8rem;
        overflow: scroll;
        margin-top: 10px;
        background-color: transparent;
    }
    #markdown {
        resize: none;
        font-family: monospace;
        padding: .5rem;
    }
    #html {
        padding: 1rem;
    }
    #html-wrapper {
        position: relative;
    }
    .h2-container {
        display: flex;
        justify-content: space-between;
        align-items: center;
    }
    #copy {
        /* right bottom corner */
        position: absolute;
        bottom: 30px;
        right: 30px;
    }
</style>

<script>

function loadScript(url, callback) {
    // Adding the script tag to the head as suggested before
    var head = document.getElementsByTagName('head')[0];
    var script = document.createElement('script');
    script.type = 'text/javascript';
    script.src = url;

    // Then bind the event to the callback function.
    // There are several events for cross browser compatibility.
    script.onreadystatechange = callback;
    script.onload = callback;

    script.onerror = function(err) {
    console.error(err);
    };

    // Fire the loading
    head.appendChild(script);
}

const debounce = (callback, wait) => {
    let timeoutId = null;
    return (...args) => {
        window.clearTimeout(timeoutId);
        timeoutId = window.setTimeout(() => {
            callback(...args);
        }, wait);
    };
}

loadScript("https://unpkg.com/showdown/dist/showdown.min.js", () => {
    var htmlElement = document.getElementById('html');
    var markdownElement = document.getElementById('markdown');
    var converter = new showdown.Converter();
    converter.setOption("tables", true);
    converter.setOption("simplifiedAutoLink", true);
    converter.setOption("disableForced4SpacesIndentedSublists", true);
    var storeDebounce = debounce((event) => {
        localStorage.setItem('markdown', markdownElement.value);
    }, 1000);
    markdownElement.addEventListener("input", debounce((event) => {
        htmlElement.innerHTML = converter.makeHtml(markdownElement.value);
        storeDebounce(event);
    }, 100), false);
    markdownElement.dispatchEvent(new Event('input'));
});

function waitForElm(selector) {
    return new Promise(resolve => {
        if (document.querySelector(selector)) {
            return resolve(document.querySelector(selector));
        }

        const observer = new MutationObserver(mutations => {
            if (document.querySelector(selector)) {
                observer.disconnect();
                resolve(document.querySelector(selector));
            }
        });

        observer.observe(document.body, {
            childList: true,
            subtree: true
        });
    });
}

function copyElement(target) {
    const content = target.innerHTML;
    const blob = new Blob([content], { type: 'text/html' });
    const clipboardItem = new window.ClipboardItem({ 'text/html': blob });
    navigator.clipboard.write([clipboardItem]);
};

waitForElm('#copy').then((elm) => {
    elm.addEventListener('click', (event) => {
        copyElement(document.getElementById('html'));
    });
});

waitForElm('#markdown').then((elm) => {
    const markdown = localStorage.getItem('markdown');
    if (markdown) {
        elm.value = markdown;
        elm.dispatchEvent(new Event('input'));
    }
});

</script>

!!! note ""
    **This is a simple Markdown to HTML converter.**

    Click the `Copy` button to copy the HTML (rich text) to your clipboard, which you can then paste into rich text editors.

<div class="container">
    <div>
        <b>Markdown</b>
        <textarea id="markdown"></textarea>
    </div>
    <div>
        <div class="b-container">
            <b>HTML</b>
        </div>    
        <div id="html-wrapper">
            <div id="html"></div>
            <button class="md-button" style="font-size:.8rem" id="copy">
                Copy
            </button>
        </div>
    </div>
</div>
