# hi2

this is a kind of editor that only support the md files 

1. 

---

```javascript
// ─────────────────────────────────────────────
// PREVIEW PANE
// ─────────────────────────────────────────────
function PreviewPane({ blocks }) {
  return (
    <div className="preview-mode-wrap">
      <div className="doc-content-area">
        {blocks.map(b => (
          <div key={b.id} className="preview-block">
            <BlockContent block={b} onChange={() => {}} onAddAfter={() => {}} isPreview/>
          </div>
        ))}
      </div>
    </div>
  );
}
```