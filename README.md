# llm-code-format

**From source code to Markdown to LLMs and back again**

A TypeScript library for parsing and serializing multiple code files in Markdown format, specifically designed for Large Language Model (LLM) interactions. The idea of this comes from the fact that when you ask an LLM to write code for you with multiple files, different models have different "preferences" when it comes to representing code. This library aims to provide a consistent way to parse and serialize code files in Markdown format, regardless of the format used by the LLM. It may also be useful for other applications that need to work with multiple code files in Markdown format, such as being able to extract code files from a Markdown document or generate a Markdown document from code files.

See also [editcodewithai](https://github.com/vizhub-core/editcodewithai), a higher level code editing tool built on `llm-code-format`.

## Features

- Parse code blocks from various Markdown formats including:

  - Bold Format (`**filename.js**`)
  - Backtick-Heading Format (`### `filename.js``)
  - Standard Heading Format (`### filename.js`)
  - Colon Format (`filename.js:`)
  - Hash Format (`# filename.js`)
  - File Bold Format (`### File: **filename.js**`)
  - Heading Bold Format (`### **filename.js**`)
  - Numbered Bold Format (`1. **filename.js**`)
  - Numbered Backtick Format (`### 1. `filename.js``)

- Streaming parser for real-time code editing views
- Serialize code files back to Markdown in a consistent format
- TypeScript support with full type definitions
- Zero dependencies
- Comprehensive test coverage

## Installation

```bash
npm install llm-code-format
```

## Usage Examples

### Basic Example Usage

```typescript
import { parseMarkdownFiles, formatMarkdownFiles } from "llm-code-format";

// Parse Markdown content containing code blocks
const markdownString = `
**index.html**

\`\`\`html
<h1>Hello World</h1>
\`\`\`

**styles.css**

\`\`\`css
body { color: blue; }
\`\`\`
`;

const { files, format } = parseMarkdownFiles(markdownString);
console.log(format); // "Bold Format"
console.log(files); // Object with filenames as keys and contents as values

// Serialize files back to Markdown
const files = {
  "index.html": "<h1>Hello World</h1>",
  "styles.css": "body { color: blue; }",
};

const markdown = formatMarkdownFiles(files);
```

### Recipe: Extract Files from Blog Posts

This "recipe" shows how to extract example code from blog posts such that they can be run (works with Astro):

```js
import { parseMarkdownFiles } from "llm-code-format";
import fs from "fs";
import path from "path";

const blogDir = "src/pages/blog";
const publicDir = "public/examples";

// Read all markdown files from blog directory
fs.readdirSync(blogDir)
  .filter((fileName) => fileName.endsWith(".md"))
  .forEach((fileName) => {
    const postName = fileName.replace(".md", "");
    const filePath = path.join(blogDir, fileName);
    const markdownString = fs.readFileSync(filePath, "utf8");

    // Use llm-code-format to extract code blocks from markdown!
    const { files } = parseMarkdownFiles(markdownString);

    if (Object.keys(files).length === 0) return;
    const outputDirectory = path.join(publicDir, postName);
    if (fs.existsSync(outputDirectory)) {
      fs.rmSync(outputDirectory, { recursive: true });
    }
    fs.mkdirSync(outputDirectory);
    Object.entries(files).forEach(([name, content]) => {
      fs.writeFileSync(path.join(outputDirectory, name), content);
    });
  });
```

## API

### parseMarkdownFiles(markdownString: string, format?: string)

Parses a Markdown string containing code blocks and returns an object with:

- `files`: Object with filenames as keys and file contents as values
- `format`: String indicating the detected format

Optional `format` parameter to specify a particular format to parse:

- 'backtick-heading'
- 'file-bold'
- 'numbered-backtick'
- 'heading-bold'
- 'standard-heading'
- 'colon'
- 'bold'
- 'hash'
- 'numbered-bold'

### formatMarkdownFiles(files: FileCollection)

Converts a FileCollection object into a Markdown string using the Bold Format.

### StreamingMarkdownParser

A streaming parser that processes Markdown content chunk by chunk, emitting callbacks when file names or code blocks are detected. This is particularly useful for real-time code editing views where users can see LLMs edit specific files as they generate content.

````typescript
import {
  StreamingMarkdownParser,
  StreamingParserCallbacks,
} from "llm-code-format";

// Define callbacks for file name changes, code lines, and non-code lines
const callbacks: StreamingParserCallbacks = {
  onFileNameChange: (fileName, format) => {
    console.log(`File changed to: ${fileName} (${format})`);
    // Update UI to show the current file being edited
  },
  onCodeLine: (line) => {
    console.log(`Code line: ${line}`);
    // Append the line to the current file's content in the UI
  },
  onNonCodeLine: (line) => {
    console.log(`Comment/text: ${line}`);
    // Process non-code, non-header lines (e.g., for displaying comments)
  },
};

// Create a new parser instance with the callbacks
const parser = new StreamingMarkdownParser(callbacks);

// Process chunks as they arrive (e.g., from an LLM streaming response)
parser.processChunk("**index.html**\n");
parser.processChunk("```html\n");
parser.processChunk("<h1>Hello World</h1>\n");
parser.processChunk("</html>\n");
parser.processChunk("```\n");

// Flush any remaining content when the stream ends
parser.flushRemaining();
````

#### Methods

- **constructor(callbacks: StreamingParserCallbacks)**: Creates a new parser instance with the specified callbacks.
- **processChunk(chunk: string)**: Processes a chunk of text from the stream.
- **flushRemaining()**: Processes any remaining content in the buffer after the stream has ended.

#### Callback Interface

```typescript
type StreamingParserCallbacks = {
  onFileNameChange: (fileName: string, format: string) => void;
  onCodeLine: (line: string) => void;
  onNonCodeLine?: (line: string) => void;
};
```

- **onFileNameChange**: Called when a file header is detected outside a code fence.
- **onCodeLine**: Called for each line emitted from inside a code fence.
- **onNonCodeLine**: Called for each line outside code fences that is not a file header.

Currently, the streaming parser only supports the "Bold Format" (`**filename.js**`), but is designed to be extensible for supporting additional formats in the future.

## Contributing

Contributions welcome! Please read the [CONTRIBUTING.md](CONTRIBUTING.md) guidelines before submitting a PR. All changes should start with creating an issue first.
