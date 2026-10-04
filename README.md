# @glrk/react-flipbook-tts

A React flipbook that reads its pages aloud, highlights each word as it is spoken, and turns the page by itself. Works in React and Next.js, with support for custom voices such as Azure or Kokoro.

> **Status:** in development. The API below is the planned v1 and may change before `1.0.0`.

<!-- Add a short GIF here: a page flipping while words highlight -->

## Features

- **Page flip:** click, swipe or use the arrow keys to turn pages
- **Read aloud:** uses the browser's built-in speech, with no API key needed
- **Word highlighting:** the current word is highlighted in time with the voice
- **Auto page turn:** the book turns the page when it finishes reading
- **Custom voices:** plug in Azure, Kokoro or any other text-to-speech service
- **TypeScript types included:** works in JavaScript projects too

## Works with

| Setup | Supported |
| --- | --- |
| React (JavaScript) | Yes |
| React + TypeScript | Yes |
| Next.js App Router | Yes, no `"use client"` needed in your file |
| Next.js Pages Router | Yes |

Requires React 18 or newer.

## Install

```bash
npm install @glrk/react-flipbook-tts
```

## Quick start

```jsx
import { FlipBook, Page, Readable } from "@glrk/react-flipbook-tts";

export default function Story() {
  return (
    <FlipBook
      width={400}
      height={560}
      readAloud={{ lang: "en-IN", autoFlip: true, highlight: "word" }}
    >
      <Page>
        <img src="/cover.png" alt="The Little Rocket" />
      </Page>
      <Page>
        <Readable>Once upon a time, a little rocket wanted to see the moon.</Readable>
      </Page>
      <Page>
        <Readable>So it packed a sandwich and counted down from ten.</Readable>
      </Page>
    </FlipBook>
  );
}
```

Text inside `<Readable>` is read aloud and highlighted. Anything else on a page, such as images, is shown but not read.

## Components

### `<FlipBook>`

| Prop | Type | Default | Description |
| --- | --- | --- | --- |
| `width` | `number` | required | Width of one page in pixels |
| `height` | `number` | required | Height of one page in pixels |
| `display` | `"single" \| "double" \| "auto"` | `"auto"` | One page, two pages, or decided by screen width |
| `page` | `number` | | Current page, when you control it yourself |
| `defaultPage` | `number` | `0` | Starting page, when the book controls it |
| `onPageChange` | `(page: number) => void` | | Called after each page change |
| `flipDuration` | `number` | `600` | Flip animation time in milliseconds |
| `swipe` | `boolean` | `true` | Allow drag and touch to flip |
| `readAloud` | `boolean \| ReadAloudOptions` | `false` | Turn on read aloud, or pass options |
| `className` | `string` | | Class for the book container |
| `style` | `CSSProperties` | | Inline style for the book container |

### `readAloud` options

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `lang` | `string` | browser default | Language code, such as `"en-IN"` |
| `rate` | `number` | `1` | Speaking speed, from `0.5` to `2` |
| `voice` | `string` | | Voice name, passed to the voice engine |
| `engine` | `SpeechEngine` | browser speech | Custom voice engine (see below) |
| `autoFlip` | `boolean` | `false` | Turn the page when reading ends |
| `autoFlipDelay` | `number` | `800` | Pause before the automatic turn, in milliseconds |
| `stopOnManualFlip` | `boolean` | `true` | Stop reading when the user flips by hand |
| `highlight` | `"word" \| "sentence" \| "none"` | `"word"` | What to highlight while reading |
| `highlightClassName` | `string` | | Your own class for the highlighted text |
| `onStart` | `(page: number) => void` | | Called when reading starts |
| `onEnd` | `(page: number) => void` | | Called when a page finishes reading |
| `onError` | `(error: unknown) => void` | | Called when speech fails |

### `<Page>`

| Prop | Type | Description |
| --- | --- | --- |
| `children` | `ReactNode` | Page content |
| `text` | `string` | Text to read instead of the `<Readable>` content |
| `skipReadAloud` | `boolean` | Do not read this page |
| `className` | `string` | Class for the page |

### `<Readable>`

| Prop | Type | Default | Description |
| --- | --- | --- | --- |
| `children` | `string` | required | The text to show, read and highlight |
| `as` | HTML tag name | `"p"` | Element to render |

## Hooks

### `useFlipBook()`

Control the book from any component inside `<FlipBook>`.

```jsx
import { useFlipBook } from "@glrk/react-flipbook-tts";

function PageCounter() {
  const { page, totalPages, next, prev } = useFlipBook();
  return (
    <div>
      <button onClick={prev}>Back</button>
      Page {page + 1} of {totalPages}
      <button onClick={next}>Next</button>
    </div>
  );
}
```

Returns `page`, `totalPages`, `isFlipping`, `next()`, `prev()`, `goTo(page)`, `play()`, `pause()`, `resume()` and `stop()`.

### `useReadAloud(options)`

Read any text aloud, with or without a flipbook.

```jsx
import { useReadAloud } from "@glrk/react-flipbook-tts";

function SpeakButton() {
  const { speak, stop, status } = useReadAloud({ lang: "en-IN" });
  return status === "playing"
    ? <button onClick={stop}>Stop</button>
    : <button onClick={() => speak("Hello there!")}>Speak</button>;
}
```

Returns `status` (`"idle"`, `"playing"` or `"paused"`), `isSupported`, `wordIndex`, `speak(text)`, `pause()`, `resume()` and `stop()`.

## Custom voices

By default the book uses the browser's built-in voices. To use another service, write one function that turns text into audio and pass it to `createAudioEngine`.

```jsx
import { FlipBook, createAudioEngine } from "@glrk/react-flipbook-tts";

const engine = createAudioEngine({
  async synthesize({ text, lang, voice, signal }) {
    const res = await fetch("/api/tts", {
      method: "POST",
      body: JSON.stringify({ text, lang, voice }),
      signal,
    });
    return { audio: await res.blob() };
  },
});

<FlipBook width={400} height={560} readAloud={{ engine, voice: "en-IN-NeerjaNeural" }}>
  {/* pages */}
</FlipBook>
```

Create the engine once, outside your component.

**Keep your API key on the server.** `synthesize` should call your own backend route, such as a Next.js route handler, which holds the key and returns the audio. Never put a provider key in browser code.

**Word highlighting with custom voices:** if your service returns word timings, return them as `marks` (a list of `{ time, charIndex }`, with `time` in seconds) and words highlight in sync. Without `marks`, the book highlights one sentence at a time.

## Browser notes

- **Voices differ per device.** The same `lang` can sound different on Android, iOS and desktop. Use a custom voice engine if you need the same voice everywhere.
- **iOS Safari needs a tap.** Speech only starts after the user taps something, so reading cannot begin by itself on page load.
- **Some Android voices do not report word positions.** The book falls back to sentence highlighting on those voices.

<!-- Add the tested browser table here after the device check -->

## Roadmap

- `useVoices` hook to list available voices
- `renderControls` prop for a fully custom control bar
- Right-to-left books and cover pages
- Recipes for Azure and Kokoro

## Acknowledgements

Built with [page-flip](https://www.npmjs.com/package/page-flip) and [react-text-to-speech](https://www.npmjs.com/package/react-text-to-speech).

## Licence

MIT
