# Viewers

A collection of standalone HTML+CSS+JS viewers for various file formats.

## Overview

This project provides lightweight, browser-based viewers that can be deployed on any web server. Each viewer is self-contained and runs directly in the browser without a build process.

## Supported Formats

| Viewer | File | Description |
|--------|------|-------------|
| tiaga.html | Markdown (.md) | Markdown document viewer |
| md.html | Markdown (.md) | Simple Markdown renderer |
| bus.html | - | Hong Kong bus ETA display |
| chat.html | JSON | JSON chat history renderer |
| chattree.html | JSON | JSON chat tree explorer with collapsible hierarchy |
| json.html | JSON | Interactive JSON tree viewer with search |
| mermaid.html | - | Mermaid diagram editor with live preview |
| Taiga Viewer | Taiga dump files | Viewer for Taiga project exports |
| Calendar | Calendar files | Calendar display viewer |

## Usage

Open any viewer file directly in a browser:

```
https://your-server/path/to/viewers/tiaga.html
```

Or specify a file to view via query parameter:

```
https://your-server/path/to/viewers/tiaga.md?file=/path/to/document.md
```

## API Integration

If `getAnyURL.php` exists at the root of the host, viewers can fetch files via:

```
https://your-host/viewer-name.html?url=/path/to/remote-file
```

## Development

All viewers are standalone HTML files with embedded CSS and JavaScript. No build step required.

## License

MIT