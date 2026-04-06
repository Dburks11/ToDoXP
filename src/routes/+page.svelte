<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import { EditorContent, Editor } from "svelte-tiptap";
  import StarterKit from "@tiptap/starter-kit";
  import Underline from "@tiptap/extension-underline";
  import { TextStyle } from "@tiptap/extension-text-style";
  import FontSize from "tiptap-extension-font-size";
  import type { AnyExtension } from "@tiptap/core";
  import { getCurrentWindow } from "@tauri-apps/api/window";
  import "../app.css";

  let editor = $state<Editor>();
  let fontSize = $state(16);
  let pinned = $state(true);

  onMount(() => {
    editor = new Editor({
      element: document.createElement("div"),
      extensions: [StarterKit, Underline, TextStyle, FontSize as AnyExtension],
      content: localStorage.getItem("todoxp") ?? "",
      onUpdate({ editor: e }) {
        localStorage.setItem("todoxp", e.getHTML());
      },
      onTransaction() {
        editor = editor;
      },
    });
  });

  onDestroy(() => {
    editor?.destroy();
  });

  async function togglePin() {
    pinned = !pinned;
    await getCurrentWindow().setAlwaysOnTop(pinned);
  }

  function changeFontSize(delta: number) {
    if (!editor) return;
    fontSize = Math.max(8, Math.min(72, fontSize + delta));
    editor.chain().focus().setFontSize(`${fontSize}px`).run();
  }

  function handleKeydown(ev: KeyboardEvent) {
    if ((ev.ctrlKey || ev.metaKey) && (ev.key === "=" || ev.key === "+")) {
      ev.preventDefault();
      changeFontSize(2);
    } else if ((ev.ctrlKey || ev.metaKey) && ev.key === "-") {
      ev.preventDefault();
      changeFontSize(-2);
    }
  }
</script>

<svelte:window onkeydown={handleKeydown} />

<div class="app">
  <div class="header">
    <h1>To Do</h1>
    <button
      class="pin-btn"
      class:pinned
      onclick={togglePin}
      title={pinned ? "Unpin window" : "Pin window"}
    >
      <svg width="16" height="16" viewBox="0 0 16 16" fill="currentColor">
        <path
          d="M4.146.146A.5.5 0 0 1 4.5 0h7a.5.5 0 0 1 .5.5c0 .68-.342 1.174-.646 1.479-.126.125-.25.224-.354.298v4.431l.078.048c.203.127.476.314.751.555C12.36 7.775 13 8.527 13 9.5a.5.5 0 0 1-.5.5H8.5v5.5a.5.5 0 0 1-1 0V10H3.5a.5.5 0 0 1-.5-.5c0-.973.64-1.725 1.17-2.189A6 6 0 0 1 5 6.708V2.277a3 3 0 0 1-.354-.298C4.342 1.674 4 1.179 4 .5a.5.5 0 0 1 .146-.354"
        />
      </svg>
    </button>
  </div>
  {#if editor}
    <EditorContent {editor} />
  {/if}
</div>
