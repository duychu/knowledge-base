---
sidebar_position: 1
---

# Homelab

## Test

```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```

```jsx title="/src/components/HelloCodeTitle.js" showLineNumbers
function HelloCodeTitle(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

```js title="/src/components/HelloCodeTitle.js" showLineNumbers
function HighlightSomeText(highlight) {
  if (highlight) {
    // highlight-next-line
    return 'This text is highlighted!';
  }

  return 'Nothing highlighted';
}

function HighlightMoreText(highlight) {
  // highlight-start
  if (highlight) {
    return 'This range is highlighted!';
  }
  // highlight-end

  return 'Nothing highlighted';
}
```