<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import { EditorContent, Editor } from "svelte-tiptap";
  import StarterKit from "@tiptap/starter-kit";
  import Underline from "@tiptap/extension-underline";
  import { TextStyle } from "@tiptap/extension-text-style";
  import FontSize from "tiptap-extension-font-size";
  import type { AnyExtension } from "@tiptap/core";
  import "../app.css";

  let editor = $state<Editor>();
  let fontSize = $state(16);

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
  <h1>To Do</h1>
  {#if editor}
    <EditorContent {editor} />
  {/if}
</div>
