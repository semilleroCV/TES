# Documentation

This is another example page showing how to structure documentation.

## Getting Started

Welcome to the documentation section. You can add:

- Guides
- Tutorials  
- API references
- Code examples

## Example Chart

```js
const data = Array.from({length: 20}, (_, i) => ({
  x: i,
  y: Math.sin(i / 3) * 10 + 20
}));
```

```js
Plot.plot({
  marks: [
    Plot.line(data, {x: "x", y: "y", stroke: "steelblue"}),
    Plot.dot(data, {x: "x", y: "y", fill: "steelblue"})
  ],
  grid: true
})
```

---

[← Back to Home](/)

