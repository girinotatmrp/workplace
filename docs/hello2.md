1. hi this is a kind of hight test so bascally we dont need this kind of this kok hope you une
2. 

```javascript
import { useState, useEffect, useRef, useCallback, useMemo } from 'react';
import { docsApi } from '../services/workspaceApi';
import toast from 'react-hot-toast';
import {
  Save, RefreshCw, Download, Eye, Edit3, Columns, Bold, Italic,
  Strikethrough, Highlighter, Heading1, Heading2, Heading3, List,
  ListOrdered, Quote, Minus, CheckSquare, X, Type, Code, Link,
  Image, Table, AlertCircle, Superscript, Subscript, Copy, Trash2,
  Plus, ChevronDown, Info, Lightbulb, TriangleAlert, Zap, Flame,
  GripVertical, Hash, FileCode, ArrowRight,
} from 'lucide-react';

// ═══════════════════════════════════════════════════════════════════
// BLOCK TYPES & SCHEMA
// Each block has: id, type, attrs, content (for text blocks: rich spans)
// ═══════════════════════════════════════════════════════════════════

const BLOCK_TYPES = {
  PARAGRAPH:   'paragraph',
  H1:          'h1',
  H2:          'h2',
  H3:          'h3',
  H4:          'h4',
  BLOCKQUOTE:  'blockquote',
  CODE_BLOCK:  'code_block',
  BULLET_LIST: 'bullet_list',
  ORDERED_LIST:'ordered_list',
  TASK_LIST:   'task_list',
  HR:          'hr',
  TABLE:       'table',
  IMAGE:       'image',
  LINK:        'link',
  ALERT:       'alert',
};

// Text blocks — can contain inline formatting
const TEXT_BLOCKS = new Set([
  BLOCK_TYPES.PARAGRAPH, BLOCK_TYPES.H1, BLOCK_TYPES.H2,
  BLOCK_TYPES.H3, BLOCK_TYPES.H4, BLOCK_TYPES.BLOCKQUOTE,
  BLOCK_TYPES.BULLET_LIST, BLOCK_TYPES.ORDERED_LIST, BLOCK_TYPES.TASK_LIST,
]);

// Void blocks — no inline formatting
const VOID_BLOCKS = new Set([
  BLOCK_TYPES.HR, BLOCK_TYPES.TABLE, BLOCK_TYPES.IMAGE,
  BLOCK_TYPES.CODE_BLOCK, BLOCK_TYPES.LINK, BLOCK_TYPES.ALERT,
]);

// ─── ID generator ─────────────────────────────────────────────────
let _idCount = 0;
const genId = () => `blk_${Date.now()}_${++_idCount}`;

// ─── Default block factories ──────────────────────────────────────
const makeBlock = (type, attrs = {}, text = '') => ({
  id: genId(), type, attrs,
  text,         // raw text for simple text blocks
  items: [],    // for lists
  rows: [],     // for tables
});

const defaultBlock = () => makeBlock(BLOCK_TYPES.PARAGRAPH, {}, '');

// ═══════════════════════════════════════════════════════════════════
// MARKDOWN ↔ BLOCKS CONVERSION
// ═══════════════════════════════════════════════════════════════════

const mdToBlocks = (md = '') => {
  if (!md.trim()) return [defaultBlock()];
  const lines = md.split('\n');
  const blocks = [];
  let i = 0;

  const flush = () => {};

  while (i < lines.length) {
    const line = lines[i];

    // Skip blank
    if (!line.trim()) { i++; continue; }

    // Fenced code block
    if (line.startsWith('```')) {
      const lang = line.slice(3).trim();
      const codeLines = [];
      i++;
      while (i < lines.length && !lines[i].startsWith('```')) {
        codeLines.push(lines[i]);
        i++;
      }
      i++; // skip closing ```
      blocks.push(makeBlock(BLOCK_TYPES.CODE_BLOCK, { lang }, codeLines.join('\n')));
      continue;
    }

    // GH Alert
    const alertMatch = line.match(/^> \[!(NOTE|TIP|WARNING|IMPORTANT|CAUTION)\]/);
    if (alertMatch) {
      const type = alertMatch[1];
      const bodyLines = [];
      i++;
      while (i < lines.length && lines[i].startsWith('> ')) {
        bodyLines.push(lines[i].slice(2));
        i++;
      }
      blocks.push(makeBlock(BLOCK_TYPES.ALERT, { alertType: type }, bodyLines.join('\n')));
      continue;
    }

    // HR
    if (/^(-{3,}|\*{3,}|_{3,})$/.test(line.trim())) {
      blocks.push(makeBlock(BLOCK_TYPES.HR));
      i++; continue;
    }

    // Headings
    const hMatch = line.match(/^(#{1,4}) (.+)$/);
    if (hMatch) {
      const level = hMatch[1].length;
      const type = ['h1','h2','h3','h4'][level-1];
      blocks.push(makeBlock(BLOCK_TYPES[type.toUpperCase()], {}, hMatch[2]));
      i++; continue;
    }

    // Image
    const imgMatch = line.match(/^!\[([^\]]*)\]\(([^)]+)\)$/);
    if (imgMatch) {
      blocks.push(makeBlock(BLOCK_TYPES.IMAGE, { alt: imgMatch[1], src: imgMatch[2] }));
      i++; continue;
    }

    // Standalone link block
    const lnkMatch = line.match(/^\[([^\]]+)\]\(([^)]+)\)$/);
    if (lnkMatch) {
      blocks.push(makeBlock(BLOCK_TYPES.LINK, { text: lnkMatch[1], href: lnkMatch[2] }));
      i++; continue;
    }

    // Table
    if (line.startsWith('|') && lines[i+1]?.match(/^\|[-| :]+\|/)) {
      const parseRow = r => r.trim().replace(/^\||\|$/g,'').split('|').map(c => c.trim());
      const headers = parseRow(line);
      i += 2; // skip separator
      const rows = [headers];
      while (i < lines.length && lines[i].startsWith('|')) {
        rows.push(parseRow(lines[i]));
        i++;
      }
      blocks.push(makeBlock(BLOCK_TYPES.TABLE, { headers: rows[0] }, '', ));
      blocks[blocks.length-1].rows = rows;
      continue;
    }

    // Task list
    if (/^- \[[ x]\] /.test(line)) {
      const items = [];
      while (i < lines.length && /^- \[[ x]\] /.test(lines[i])) {
        const done = lines[i][3] === 'x';
        items.push({ done, text: lines[i].slice(6) });
        i++;
      }
      const b = makeBlock(BLOCK_TYPES.TASK_LIST);
      b.items = items;
      blocks.push(b);
      continue;
    }

    // Unordered list
    if (/^[-*+] /.test(line)) {
      const items = [];
      while (i < lines.length && /^[-*+] /.test(lines[i])) {
        items.push({ text: lines[i].slice(2) });
        i++;
      }
      const b = makeBlock(BLOCK_TYPES.BULLET_LIST);
      b.items = items;
      blocks.push(b);
      continue;
    }

    // Ordered list
    if (/^\d+\. /.test(line)) {
      const items = [];
      while (i < lines.length && /^\d+\. /.test(lines[i])) {
        items.push({ text: lines[i].replace(/^\d+\. /, '') });
        i++;
      }
      const b = makeBlock(BLOCK_TYPES.ORDERED_LIST);
      b.items = items;
      blocks.push(b);
      continue;
    }

    // Blockquote
    if (line.startsWith('> ')) {
      const bqLines = [];
      while (i < lines.length && lines[i].startsWith('> ')) {
        bqLines.push(lines[i].slice(2));
        i++;
      }
      blocks.push(makeBlock(BLOCK_TYPES.BLOCKQUOTE, {}, bqLines.join(' ')));
      continue;
    }

    // Paragraph
    const paraLines = [];
    while (i < lines.length && lines[i].trim() && !lines[i].match(/^(#{1,4} |> |\`\`\`|!?\[|[-*+] |\d+\. |---|===|\|)/)) {
      paraLines.push(lines[i]);
      i++;
    }
    if (paraLines.length) {
      blocks.push(makeBlock(BLOCK_TYPES.PARAGRAPH, {}, paraLines.join('\n')));
    } else {
      i++;
    }
  }

  return blocks.length ? blocks : [defaultBlock()];
};

const blocksToMd = (blocks) => {
  return blocks.map(b => {
    switch (b.type) {
      case BLOCK_TYPES.PARAGRAPH:   return b.text;
      case BLOCK_TYPES.H1:          return `# ${b.text}`;
      case BLOCK_TYPES.H2:          return `## ${b.text}`;
      case BLOCK_TYPES.H3:          return `### ${b.text}`;
      case BLOCK_TYPES.H4:          return `#### ${b.text}`;
      case BLOCK_TYPES.BLOCKQUOTE:  return b.text.split('\n').map(l => `> ${l}`).join('\n');
      case BLOCK_TYPES.CODE_BLOCK:  return `\`\`\`${b.attrs.lang || ''}\n${b.text}\n\`\`\``;
      case BLOCK_TYPES.HR:          return '---';
      case BLOCK_TYPES.IMAGE:       return `![${b.attrs.alt || ''}](${b.attrs.src || ''})`;
      case BLOCK_TYPES.LINK:        return `[${b.attrs.text || 'Link'}](${b.attrs.href || '#'})`;
      case BLOCK_TYPES.ALERT:
        return `> [!${b.attrs.alertType}]\n${b.text.split('\n').map(l => `> ${l}`).join('\n')}`;
      case BLOCK_TYPES.BULLET_LIST:
        return b.items.map(it => `- ${it.text}`).join('\n');
      case BLOCK_TYPES.ORDERED_LIST:
        return b.items.map((it, idx) => `${idx+1}. ${it.text}`).join('\n');
      case BLOCK_TYPES.TASK_LIST:
        return b.items.map(it => `- [${it.done ? 'x' : ' '}] ${it.text}`).join('\n');
      case BLOCK_TYPES.TABLE:
        if (!b.rows?.length) return '';
        const [header, ...rest] = b.rows;
        const sep = '| ' + header.map(() => '---').join(' | ') + ' |';
        const fmt = r => '| ' + r.join(' | ') + ' |';
        return [fmt(header), sep, ...rest.map(fmt)].join('\n');
      default: return b.text || '';
    }
  }).join('\n\n');
};

// ═══════════════════════════════════════════════════════════════════
// INLINE MARKDOWN RENDER (spans only — no block elements)
// ═══════════════════════════════════════════════════════════════════

const renderInline = (text = '') => {
  if (!text) return '';
  return text
    .replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;')
    .replace(/\*\*\*(.+?)\*\*\*/g,'<strong><em>$1</em></strong>')
    .replace(/\*\*(.+?)\*\*/g,'<strong>$1</strong>')
    .replace(/__(.+?)__/g,'<strong>$1</strong>')
    .replace(/\*(.+?)\*/g,'<em>$1</em>')
    .replace(/_([^_\n]+)_/g,'<em>$1</em>')
    .replace(/~~(.+?)~~/g,'<s>$1</s>')
    .replace(/==(.+?)==/g,'<mark>$1</mark>')
    .replace(/\^([^\^\n]+)\^/g,'<sup>$1</sup>')
    .replace(/~([^~\n]+)~/g,'<sub>$1</sub>')
    .replace(/`([^`]+)`/g,(_, c) => `<code>${c.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;')}</code>`)
    .replace(/!\[([^\]]*)\]\(([^)]+)\)/g,'<img src="$2" alt="$1" style="max-width:100%;border-radius:.4rem" />')
    .replace(/\[([^\]]+)\]\(([^)]+)\)/g,'<a href="$2" target="_blank" rel="noopener noreferrer">$1</a>')
    .replace(/(?<!["\(])(https?:\/\/[^\s<>"]+)/g,'<a href="$1" target="_blank" rel="noopener noreferrer">$1</a>');
};

// ═══════════════════════════════════════════════════════════════════
// ALERT CONFIG
// ═══════════════════════════════════════════════════════════════════

const ALERT_CONFIG = {
  NOTE:      { icon: Info,          label: 'Note',      color: '#3b82f6', bg: '#eff6ff', darkBg: '#1e3a5f', darkColor: '#93c5fd' },
  TIP:       { icon: Lightbulb,     label: 'Tip',       color: '#10b981', bg: '#f0fdf4', darkBg: '#14532d', darkColor: '#6ee7b7' },
  WARNING:   { icon: TriangleAlert, label: 'Warning',   color: '#f59e0b', bg: '#fffbeb', darkBg: '#451a03', darkColor: '#fcd34d' },
  IMPORTANT: { icon: Zap,           label: 'Important', color: '#8b5cf6', bg: '#f5f3ff', darkBg: '#2e1065', darkColor: '#c4b5fd' },
  CAUTION:   { icon: Flame,         label: 'Caution',   color: '#ef4444', bg: '#fef2f2', darkBg: '#450a0a', darkColor: '#fca5a5' },
};

// ═══════════════════════════════════════════════════════════════════
// BLOCK OVERLAY — copy/delete controls shown on hover/focus
// ═══════════════════════════════════════════════════════════════════

const BlockOverlay = ({ onCopy, onDelete }) => (
  <div className="block-overlay">
    <button className="bov-btn" onMouseDown={e => { e.preventDefault(); e.stopPropagation(); onCopy(); }} title="Copy block">
      <Copy size={11} />
    </button>
    <button className="bov-btn bov-del" onMouseDown={e => { e.preventDefault(); e.stopPropagation(); onDelete(); }} title="Delete block">
      <Trash2 size={11} />
    </button>
  </div>
);

// ═══════════════════════════════════════════════════════════════════
// FORMATTING TOOLBAR — shown only for text blocks with selection
// ═══════════════════════════════════════════════════════════════════

const FMT_BTNS = [
  { label: 'Bold **',        syntax: ['**','**'],      icon: Bold },
  { label: 'Italic *',       syntax: ['*','*'],         icon: Italic },
  { label: 'Strikethrough',  syntax: ['~~','~~'],       icon: Strikethrough },
  { label: 'Highlight ==',   syntax: ['==','=='],       icon: Highlighter },
  { label: 'Inline code `',  syntax: ['`','`'],         icon: Code },
  { label: 'Superscript ^',  syntax: ['^','^'],         icon: Superscript },
  { label: 'Subscript ~',    syntax: ['~','~'],         icon: Subscript },
];

const FormattingBar = ({ position, onFormat, onClose }) => {
  const ref = useRef(null);
  const [pos, setPos] = useState(null);

  useEffect(() => {
    if (!ref.current) return;
    const w = ref.current.offsetWidth;
    const h = ref.current.offsetHeight;
    let x = position.x - w/2;
    let y = position.y - h - 10;
    x = Math.max(8, Math.min(x, window.innerWidth - w - 8));
    if (y < 8) y = position.y + 28;
    setPos({ x, y });
  }, [position]);

  return (
    <div ref={ref} className="fmt-bar"
      style={{ top: pos?.y ?? -999, left: pos?.x ?? -999, visibility: pos ? 'visible' : 'hidden' }}
      onMouseDown={e => e.preventDefault()}>
      {FMT_BTNS.map(b => (
        <button key={b.label} className="fmt-btn" title={b.label}
          onMouseDown={e => { e.preventDefault(); onFormat(b.syntax); }}>
          <b.icon size={12} />
        </button>
      ))}
      <div className="fmt-sep" />
      <button className="fmt-btn fmt-close" title="Close" onMouseDown={e => { e.preventDefault(); onClose(); }}>
        <X size={11} />
      </button>
    </div>
  );
};

// ═══════════════════════════════════════════════════════════════════
// ELEMENT INSERTION MENU
// ═══════════════════════════════════════════════════════════════════

const ELEMENTS = [
  { type: BLOCK_TYPES.H1,          label: 'Heading 1',     icon: Heading1,    group: 'Text' },
  { type: BLOCK_TYPES.H2,          label: 'Heading 2',     icon: Heading2,    group: 'Text' },
  { type: BLOCK_TYPES.H3,          label: 'Heading 3',     icon: Heading3,    group: 'Text' },
  { type: BLOCK_TYPES.BLOCKQUOTE,  label: 'Blockquote',    icon: Quote,       group: 'Text' },
  { type: BLOCK_TYPES.BULLET_LIST, label: 'Bullet List',   icon: List,        group: 'List' },
  { type: BLOCK_TYPES.ORDERED_LIST,label: 'Numbered List', icon: ListOrdered, group: 'List' },
  { type: BLOCK_TYPES.TASK_LIST,   label: 'Task List',     icon: CheckSquare, group: 'List' },
  { type: BLOCK_TYPES.CODE_BLOCK,  label: 'Code Block',    icon: FileCode,    group: 'Code' },
  { type: BLOCK_TYPES.IMAGE,       label: 'Image',         icon: Image,       group: 'Media' },
  { type: BLOCK_TYPES.LINK,        label: 'Link Block',    icon: Link,        group: 'Media' },
  { type: BLOCK_TYPES.TABLE,       label: 'Table',         icon: Table,       group: 'Data' },
  { type: BLOCK_TYPES.HR,          label: 'Divider',       icon: Minus,       group: 'Data' },
  { type: BLOCK_TYPES.ALERT,       label: 'Alert/Callout', icon: AlertCircle, group: 'Data' },
];

const ElementMenu = ({ position, onInsert, onClose }) => {
  const groups = [...new Set(ELEMENTS.map(e => e.group))];
  return (
    <div className="el-menu" style={{ top: position.y, left: position.x }}
      onMouseDown={e => e.stopPropagation()}>
      <div className="el-menu-header">
        <span>Insert element</span>
        <button className="el-close" onClick={onClose}><X size={12}/></button>
      </div>
      {groups.map(g => (
        <div key={g}>
          <div className="el-group-label">{g}</div>
          {ELEMENTS.filter(e => e.group === g).map(el => (
            <button key={el.type} className="el-item"
              onMouseDown={e => { e.preventDefault(); onInsert(el.type); onClose(); }}>
              <el.icon size={13} className="el-item-icon" />
              <span>{el.label}</span>
            </button>
          ))}
        </div>
      ))}
    </div>
  );
};

// ═══════════════════════════════════════════════════════════════════
// BLOCK EDITORS — one component per block type
// ═══════════════════════════════════════════════════════════════════

// Text block (paragraph, h1-h4, blockquote)
const TextBlockEditor = ({ block, focused, onChange, onFocus, onBlur, onKeyDown }) => {
  const ref = useRef(null);
  useEffect(() => { if (focused && ref.current && document.activeElement !== ref.current) ref.current.focus(); }, [focused]);

  const Tag = block.type === BLOCK_TYPES.PARAGRAPH ? 'p'
            : block.type === BLOCK_TYPES.BLOCKQUOTE ? 'blockquote'
            : block.type;

  return (
    <div
      ref={ref}
      contentEditable
      suppressContentEditableWarning
      className={`tblock tblock-${block.type}`}
      data-placeholder={block.type === BLOCK_TYPES.PARAGRAPH ? 'Write something…' : block.type.replace('_',' ').replace(/h(\d)/,'Heading $1')}
      dangerouslySetInnerHTML={{ __html: renderInline(block.text) }}
      onInput={e => onChange({ ...block, text: e.currentTarget.textContent })}
      onFocus={onFocus}
      onBlur={e => { onChange({ ...block, text: e.currentTarget.textContent }); onBlur?.(); }}
      onKeyDown={onKeyDown}
    />
  );
};

// Code block
const CodeBlockEditor = ({ block, onChange }) => {
  const LANGS = ['','javascript','typescript','python','rust','go','java','c','cpp','css','html','json','yaml','bash','sql','markdown','graphql','diff'];
  return (
    <div className="code-block-editor">
      <div className="code-block-header">
        <FileCode size={12} />
        <select className="code-lang-sel" value={block.attrs.lang || ''}
          onChange={e => onChange({ ...block, attrs: { ...block.attrs, lang: e.target.value } })}>
          {LANGS.map(l => <option key={l} value={l}>{l || 'plain text'}</option>)}
        </select>
      </div>
      <textarea
        className="code-block-ta"
        value={block.text}
        onChange={e => onChange({ ...block, text: e.target.value })}
        spellCheck={false}
        placeholder="// Write your code here…"
        rows={Math.max(3, (block.text.match(/\n/g)?.length || 0) + 2)}
      />
    </div>
  );
};

// List block (bullet, ordered, task)
const ListBlockEditor = ({ block, onChange }) => {
  const addItem = () => onChange({ ...block, items: [...block.items, { text: '', done: false }] });
  const updItem = (i, patch) => onChange({ ...block, items: block.items.map((it, idx) => idx === i ? { ...it, ...patch } : it) });
  const delItem = (i) => onChange({ ...block, items: block.items.filter((_,idx) => idx !== i) });

  return (
    <div className="list-editor">
      {block.items.map((it, i) => (
        <div key={i} className="list-item-row">
          {block.type === BLOCK_TYPES.TASK_LIST && (
            <input type="checkbox" checked={!!it.done} onChange={e => updItem(i, { done: e.target.checked })} className="task-cb" />
          )}
          {block.type === BLOCK_TYPES.ORDERED_LIST && (
            <span className="ol-num">{i+1}.</span>
          )}
          {block.type === BLOCK_TYPES.BULLET_LIST && (
            <span className="ul-dot">•</span>
          )}
          <input
            type="text"
            className={`list-item-input${it.done ? ' done' : ''}`}
            value={it.text}
            onChange={e => updItem(i, { text: e.target.value })}
            placeholder="List item…"
            onKeyDown={e => {
              if (e.key === 'Enter') { e.preventDefault(); addItem(); }
              if (e.key === 'Backspace' && !it.text && block.items.length > 1) { e.preventDefault(); delItem(i); }
            }}
          />
          <button className="list-del-btn" onClick={() => delItem(i)}><X size={10}/></button>
        </div>
      ))}
      <button className="list-add-btn" onClick={addItem}>
        <Plus size={11}/> Add item
      </button>
    </div>
  );
};

// Table block
const TableBlockEditor = ({ block, onChange }) => {
  const rows = block.rows?.length ? block.rows : [['Header','Header','Header'],['Cell','Cell','Cell']];
  const setCell = (r, c, val) => {
    const nr = rows.map((row, ri) => ri===r ? row.map((cell,ci) => ci===c ? val : cell) : row);
    onChange({ ...block, rows: nr });
  };
  const addRow = () => onChange({ ...block, rows: [...rows, Array(rows[0].length).fill('Cell')] });
  const addCol = () => onChange({ ...block, rows: rows.map(r => [...r, 'Cell']) });
  const delRow = (r) => { if (rows.length>1) onChange({ ...block, rows: rows.filter((_,i)=>i!==r) }); };
  const delCol = (c) => { if (rows[0].length>1) onChange({ ...block, rows: rows.map(r=>r.filter((_,i)=>i!==c)) }); };

  return (
    <div className="table-editor">
      <div className="table-wrap">
        <table className="te-table">
          {rows.map((row, ri) => (
            <tr key={ri} className={ri===0 ? 'te-head' : ''}>
              {row.map((cell, ci) => (
                <td key={ci} className={`te-cell${ri===0?' te-hcell':''}`}>
                  <input className="te-input" value={cell}
                    onChange={e => setCell(ri, ci, e.target.value)} />
                  {ri===0 && (
                    <button className="te-del-col" onClick={() => delCol(ci)} title="Delete column"><X size={9}/></button>
                  )}
                </td>
              ))}
              <td className="te-row-ctrl">
                {ri>0 && <button onClick={() => delRow(ri)} title="Delete row"><X size={9}/></button>}
              </td>
            </tr>
          ))}
        </table>
      </div>
      <div className="table-btns">
        <button className="te-add-btn" onClick={addRow}><Plus size={10}/> Row</button>
        <button className="te-add-btn" onClick={addCol}><Plus size={10}/> Column</button>
      </div>
    </div>
  );
};

// Image block
const ImageBlockEditor = ({ block, onChange }) => (
  <div className="img-editor">
    <div className="img-fields">
      <input className="img-input" placeholder="Image URL (https://…)" value={block.attrs.src || ''}
        onChange={e => onChange({ ...block, attrs: { ...block.attrs, src: e.target.value } })} />
      <input className="img-input img-alt" placeholder="Alt text" value={block.attrs.alt || ''}
        onChange={e => onChange({ ...block, attrs: { ...block.attrs, alt: e.target.value } })} />
    </div>
    {block.attrs.src && (
      <div className="img-preview-wrap">
        <img src={block.attrs.src} alt={block.attrs.alt || ''} className="img-preview"
          onError={e => { e.target.style.display='none'; }} />
      </div>
    )}
  </div>
);

// Link block
const LinkBlockEditor = ({ block, onChange }) => (
  <div className="link-editor">
    <div className="link-icon-wrap"><Link size={13}/></div>
    <div className="link-fields">
      <input className="link-input link-text" placeholder="Link display text"
        value={block.attrs.text || ''}
        onChange={e => onChange({ ...block, attrs: { ...block.attrs, text: e.target.value } })} />
      <input className="link-input link-url" placeholder="https://…"
        value={block.attrs.href || ''}
        onChange={e => onChange({ ...block, attrs: { ...block.attrs, href: e.target.value } })} />
    </div>
    {block.attrs.href && (
      <a href={block.attrs.href} target="_blank" rel="noopener noreferrer" className="link-preview">
        <ArrowRight size={11}/> Open
      </a>
    )}
  </div>
);

// Alert block
const AlertBlockEditor = ({ block, onChange }) => {
  const cfg = ALERT_CONFIG[block.attrs.alertType] || ALERT_CONFIG.NOTE;
  const IconComp = cfg.icon;
  return (
    <div className="alert-editor" data-alert-type={block.attrs.alertType}>
      <div className="alert-type-row">
        {Object.entries(ALERT_CONFIG).map(([type, c]) => {
          const IC = c.icon;
          return (
            <button key={type}
              className={`alert-type-btn${block.attrs.alertType===type?' active':''}`}
              data-type={type}
              onMouseDown={e => { e.preventDefault(); onChange({ ...block, attrs: { ...block.attrs, alertType: type } }); }}
              title={c.label}>
              <IC size={12}/> {c.label}
            </button>
          );
        })}
      </div>
      <div className="alert-body" style={{ borderLeftColor: cfg.color, backgroundColor: cfg.bg }}>
        <div className="alert-label-row" style={{ color: cfg.color }}>
          <IconComp size={13}/>
          <span className="alert-type-label">{block.attrs.alertType}</span>
        </div>
        <textarea
          className="alert-ta"
          value={block.text}
          onChange={e => onChange({ ...block, text: e.target.value })}
          placeholder={`Write your ${cfg.label.toLowerCase()} message here…`}
          rows={Math.max(2, (block.text.match(/\n/g)?.length || 0) + 2)}
        />
      </div>
    </div>
  );
};

// HR block
const HrBlockEditor = () => (
  <div className="hr-editor"><hr className="hr-visual"/></div>
);

// ═══════════════════════════════════════════════════════════════════
// SINGLE BLOCK WRAPPER — handles focus, overlay, boundary box
// ═══════════════════════════════════════════════════════════════════

const BlockWrapper = ({ block, focused, onFocus, onUpdate, onDelete, onCopy, onKeyDown, onSelectionChange }) => {
  const ref = useRef(null);
  const isVoid = VOID_BLOCKS.has(block.type);

  const handleMouseUp = useCallback(() => {
    if (!isVoid) setTimeout(onSelectionChange, 10);
  }, [isVoid, onSelectionChange]);

  const handleKeyUp = useCallback(() => {
    if (!isVoid) setTimeout(onSelectionChange, 10);
  }, [isVoid, onSelectionChange]);

  return (
    <div
      ref={ref}
      className={`block-wrap${focused?' focused':''}${isVoid?' void-block':' text-block'}`}
      data-block-id={block.id}
      onMouseDown={() => onFocus(block.id)}
      onMouseUp={handleMouseUp}
      onKeyUp={handleKeyUp}
    >
      {/* Block content */}
      {block.type === BLOCK_TYPES.PARAGRAPH   && <TextBlockEditor block={block} focused={focused} onChange={onUpdate} onKeyDown={onKeyDown} />}
      {block.type === BLOCK_TYPES.H1          && <TextBlockEditor block={block} focused={focused} onChange={onUpdate} onKeyDown={onKeyDown} />}
      {block.type === BLOCK_TYPES.H2          && <TextBlockEditor block={block} focused={focused} onChange={onUpdate} onKeyDown={onKeyDown} />}
      {block.type === BLOCK_TYPES.H3          && <TextBlockEditor block={block} focused={focused} onChange={onUpdate} onKeyDown={onKeyDown} />}
      {block.type === BLOCK_TYPES.H4          && <TextBlockEditor block={block} focused={focused} onChange={onUpdate} onKeyDown={onKeyDown} />}
      {block.type === BLOCK_TYPES.BLOCKQUOTE  && <TextBlockEditor block={block} focused={focused} onChange={onUpdate} onKeyDown={onKeyDown} />}
      {block.type === BLOCK_TYPES.BULLET_LIST && <ListBlockEditor block={block} onChange={onUpdate} />}
      {block.type === BLOCK_TYPES.ORDERED_LIST&& <ListBlockEditor block={block} onChange={onUpdate} />}
      {block.type === BLOCK_TYPES.TASK_LIST   && <ListBlockEditor block={block} onChange={onUpdate} />}
      {block.type === BLOCK_TYPES.CODE_BLOCK  && <CodeBlockEditor block={block} onChange={onUpdate} />}
      {block.type === BLOCK_TYPES.TABLE       && <TableBlockEditor block={block} onChange={onUpdate} />}
      {block.type === BLOCK_TYPES.IMAGE       && <ImageBlockEditor block={block} onChange={onUpdate} />}
      {block.type === BLOCK_TYPES.LINK        && <LinkBlockEditor block={block} onChange={onUpdate} />}
      {block.type === BLOCK_TYPES.ALERT       && <AlertBlockEditor block={block} onChange={onUpdate} />}
      {block.type === BLOCK_TYPES.HR          && <HrBlockEditor />}

      {/* Overlay (copy/delete) — shown when focused on void, or always visible on hover */}
      <BlockOverlay onCopy={onCopy} onDelete={onDelete} />
    </div>
  );
};

// ═══════════════════════════════════════════════════════════════════
// PREVIEW RENDERER
// ═══════════════════════════════════════════════════════════════════

const renderBlockPreview = (block) => {
  switch (block.type) {
    case BLOCK_TYPES.PARAGRAPH:  return <p key={block.id} dangerouslySetInnerHTML={{ __html: renderInline(block.text) || '<br/>' }} />;
    case BLOCK_TYPES.H1:         return <h1 key={block.id} dangerouslySetInnerHTML={{ __html: renderInline(block.text) }} />;
    case BLOCK_TYPES.H2:         return <h2 key={block.id} dangerouslySetInnerHTML={{ __html: renderInline(block.text) }} />;
    case BLOCK_TYPES.H3:         return <h3 key={block.id} dangerouslySetInnerHTML={{ __html: renderInline(block.text) }} />;
    case BLOCK_TYPES.H4:         return <h4 key={block.id} dangerouslySetInnerHTML={{ __html: renderInline(block.text) }} />;
    case BLOCK_TYPES.BLOCKQUOTE: return <blockquote key={block.id} dangerouslySetInnerHTML={{ __html: renderInline(block.text) }} />;
    case BLOCK_TYPES.HR:         return <hr key={block.id} />;
    case BLOCK_TYPES.CODE_BLOCK: return (
      <pre key={block.id} className="preview-code">
        <code className={`lang-${block.attrs.lang||''}`}>
          {block.text.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;')}
        </code>
        {block.attrs.lang && <span className="code-lang-badge">{block.attrs.lang}</span>}
      </pre>
    );
    case BLOCK_TYPES.IMAGE:
      return block.attrs.src ? (
        <figure key={block.id}>
          <img src={block.attrs.src} alt={block.attrs.alt||''} className="preview-img" />
          {block.attrs.alt && <figcaption>{block.attrs.alt}</figcaption>}
        </figure>
      ) : null;
    case BLOCK_TYPES.LINK:
      return (
        <p key={block.id} className="preview-link-block">
          <a href={block.attrs.href||'#'} target="_blank" rel="noopener noreferrer">
            <ArrowRight size={13}/> {block.attrs.text || block.attrs.href}
          </a>
        </p>
      );
    case BLOCK_TYPES.BULLET_LIST:
      return <ul key={block.id}>{block.items.map((it,i) => <li key={i} dangerouslySetInnerHTML={{ __html: renderInline(it.text) }} />)}</ul>;
    case BLOCK_TYPES.ORDERED_LIST:
      return <ol key={block.id}>{block.items.map((it,i) => <li key={i} dangerouslySetInnerHTML={{ __html: renderInline(it.text) }} />)}</ol>;
    case BLOCK_TYPES.TASK_LIST:
      return (
        <ul key={block.id} className="task-list">
          {block.items.map((it,i) => (
            <li key={i} className={it.done ? 'task-done' : 'task-todo'}>
              <input type="checkbox" checked={!!it.done} readOnly />
              <span dangerouslySetInnerHTML={{ __html: renderInline(it.text) }} />
            </li>
          ))}
        </ul>
      );
    case BLOCK_TYPES.TABLE: {
      if (!block.rows?.length) return null;
      const [headers, ...rows] = block.rows;
      return (
        <table key={block.id}>
          <thead><tr>{headers.map((h,i) => <th key={i}>{h}</th>)}</tr></thead>
          <tbody>{rows.map((r,i) => <tr key={i}>{r.map((c,j) => <td key={j} dangerouslySetInnerHTML={{ __html: renderInline(c) }} />)}</tr>)}</tbody>
        </table>
      );
    }
    case BLOCK_TYPES.ALERT: {
      const cfg = ALERT_CONFIG[block.attrs.alertType] || ALERT_CONFIG.NOTE;
      const IC = cfg.icon;
      return (
        <div key={block.id} className={`preview-alert preview-alert-${block.attrs.alertType?.toLowerCase()}`}>
          <div className="preview-alert-header"><IC size={14}/><strong>{block.attrs.alertType}</strong></div>
          <div dangerouslySetInnerHTML={{ __html: renderInline(block.text) }} />
        </div>
      );
    }
    default: return null;
  }
};

// ═══════════════════════════════════════════════════════════════════
// MAIN EDITOR
// ═══════════════════════════════════════════════════════════════════

const DocEditor = ({ documentPath, onClose }) => {
  const [doc, setDoc]           = useState(null);
  const [blocks, setBlocks]     = useState([defaultBlock()]);
  const [rawMd, setRawMd]       = useState('');
  const [loading, setLoading]   = useState(true);
  const [saving, setSaving]     = useState(false);
  const [dirty, setDirty]       = useState(false);
  const [mode, setMode]         = useState('rich');  // rich | write | split | preview
  const [focusedId, setFocusedId] = useState(null);

  // Formatting bar state
  const [fmtBar, setFmtBar]     = useState(null);  // { x, y, blockId, range } | null
  const fmtBlockId              = useRef(null);
  const savedRange              = useRef(null);

  // Element insertion menu
  const [elMenu, setElMenu]     = useState(null);   // { x, y, afterId } | null

  const textareaRef = useRef(null);
  const editorRef   = useRef(null);

  // ── Load ─────────────────────────────────────────────────────────
  useEffect(() => {
    setLoading(true);
    setDirty(false);
    setFocusedId(null);
    setFmtBar(null);
    setElMenu(null);
    docsApi.getByPath(documentPath)
      .then(r => {
        const c = r.data.document.content || '';
        setDoc(r.data.document);
        setRawMd(c);
        setBlocks(mdToBlocks(c));
      })
      .catch(() => toast.error('Failed to load document'))
      .finally(() => setLoading(false));
  }, [documentPath]);

  // ── Sync blocks → rawMd when in rich mode ───────────────────────
  const syncMd = useCallback((bs) => {
    const md = blocksToMd(bs);
    setRawMd(md);
    return md;
  }, []);

  const updateBlocks = useCallback((bs) => {
    setBlocks(bs);
    syncMd(bs);
    setDirty(true);
  }, [syncMd]);

  // ── Mode switch: write↔rich ──────────────────────────────────────
  const prevMode = useRef(mode);
  useEffect(() => {
    const from = prevMode.current; const to = mode;
    prevMode.current = to;
    if (from === to) return;
    if (from === 'rich' || from === 'split') {
      // Sync blocks → md for write mode
      syncMd(blocks);
    }
    if (to === 'rich' || to === 'split') {
      // Parse raw md → blocks
      if (from === 'write') {
        const bs = mdToBlocks(rawMd);
        setBlocks(bs);
      }
    }
  }, [mode]); // eslint-disable-line

  // ── Keyboard: ⌘S ─────────────────────────────────────────────────
  useEffect(() => {
    const h = e => { if ((e.metaKey || e.ctrlKey) && e.key === 's') { e.preventDefault(); if (dirty) doSave(); } };
    window.addEventListener('keydown', h);
    return () => window.removeEventListener('keydown', h);
  }, [dirty, blocks, rawMd, mode]); // eslint-disable-line

  // ── Save ─────────────────────────────────────────────────────────
  const doSave = useCallback(async () => {
    const md = mode === 'write' ? rawMd : blocksToMd(blocks);
    setSaving(true);
    try {
      await docsApi.update(documentPath, { content: md });
      await docsApi.sync(documentPath);
      setDirty(false);
      setDoc(d => d ? { ...d, updatedAt: new Date().toISOString() } : d);
      toast.success('Saved');
    } catch { toast.error('Failed to save'); }
    finally { setSaving(false); }
  }, [documentPath, blocks, rawMd, mode]);

  const doRefresh = async () => {
    try {
      const r = await docsApi.refresh(documentPath);
      const nc = r.data.document.content || '';
      setRawMd(nc);
      setBlocks(mdToBlocks(nc));
      setDirty(false);
      toast.success('Refreshed from GitHub');
    } catch { toast.error('Failed to refresh'); }
  };

  const doExport = async () => {
    try {
      const r = await docsApi.exportMarkdown(documentPath);
      const url = URL.createObjectURL(new Blob([r.data]));
      Object.assign(document.createElement('a'), { href: url, download: `${doc?.title || documentPath}.md` }).click();
      URL.revokeObjectURL(url);
      toast.success('Exported');
    } catch { toast.error('Failed to export'); }
  };

  // ── Block operations ──────────────────────────────────────────────
  const updateBlock = useCallback((updated) => {
    setBlocks(prev => { const bs = prev.map(b => b.id === updated.id ? updated : b); syncMd(bs); setDirty(true); return bs; });
  }, [syncMd]);

  const deleteBlock = useCallback((id) => {
    setBlocks(prev => {
      const bs = prev.filter(b => b.id !== id);
      const result = bs.length ? bs : [defaultBlock()];
      syncMd(result);
      setDirty(true);
      return result;
    });
    setFocusedId(null);
  }, [syncMd]);

  const copyBlock = useCallback((id) => {
    setBlocks(prev => {
      const idx = prev.findIndex(b => b.id === id);
      if (idx < 0) return prev;
      const clone = { ...JSON.parse(JSON.stringify(prev[idx])), id: genId() };
      const bs = [...prev.slice(0, idx+1), clone, ...prev.slice(idx+1)];
      syncMd(bs);
      setDirty(true);
      return bs;
    });
  }, [syncMd]);

  const insertBlockAfter = useCallback((afterId, type) => {
    const newBlock = (() => {
      switch (type) {
        case BLOCK_TYPES.BULLET_LIST:
        case BLOCK_TYPES.ORDERED_LIST:
        case BLOCK_TYPES.TASK_LIST: {
          const b = makeBlock(type);
          b.items = [{ text: '', done: false }];
          return b;
        }
        case BLOCK_TYPES.TABLE: {
          const b = makeBlock(type);
          b.rows = [['Header','Header','Header'],['Cell','Cell','Cell']];
          return b;
        }
        case BLOCK_TYPES.IMAGE: return makeBlock(type, { src: '', alt: '' });
        case BLOCK_TYPES.LINK:  return makeBlock(type, { text: 'Link', href: 'https://' });
        case BLOCK_TYPES.CODE_BLOCK: return makeBlock(type, { lang: '' }, '');
        case BLOCK_TYPES.ALERT: return makeBlock(type, { alertType: 'NOTE' }, 'Add your message here.');
        case BLOCK_TYPES.HR:    return makeBlock(type);
        default: return makeBlock(type, {}, '');
      }
    })();

    setBlocks(prev => {
      const idx = prev.findIndex(b => b.id === afterId);
      const insertAt = idx < 0 ? prev.length : idx + 1;
      const bs = [...prev.slice(0, insertAt), newBlock, ...prev.slice(insertAt)];
      syncMd(bs);
      setDirty(true);
      return bs;
    });
    setFocusedId(newBlock.id);
  }, [syncMd]);

  // ── Block keydown — Enter creates new paragraph after ─────────────
  const makeKeyDown = useCallback((block) => (e) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      const newBlk = defaultBlock();
      setBlocks(prev => {
        const idx = prev.findIndex(b => b.id === block.id);
        const bs = [...prev.slice(0, idx+1), newBlk, ...prev.slice(idx+1)];
        syncMd(bs);
        setDirty(true);
        return bs;
      });
      setFocusedId(newBlk.id);
    }
    if (e.key === 'Backspace') {
      const el = e.currentTarget;
      if (el.textContent === '' || el.innerText === '') {
        e.preventDefault();
        setBlocks(prev => {
          const idx = prev.findIndex(b => b.id === block.id);
          if (prev.length === 1) return prev;
          const bs = prev.filter(b => b.id !== block.id);
          syncMd(bs);
          setDirty(true);
          const focusIdx = Math.max(0, idx-1);
          setFocusedId(bs[focusIdx]?.id);
          return bs;
        });
      }
    }
  }, [syncMd]);

  // ── Selection change → show formatting bar ────────────────────────
  const handleSelectionChange = useCallback((blockId) => {
    const sel = window.getSelection();
    if (!sel || sel.isCollapsed || sel.rangeCount === 0) {
      setFmtBar(null);
      return;
    }
    const range = sel.getRangeAt(0);
    const rect = range.getBoundingClientRect();
    if (rect.width < 2) { setFmtBar(null); return; }
    savedRange.current = range.cloneRange();
    fmtBlockId.current = blockId;
    setFmtBar({ x: rect.left + rect.width/2, y: rect.top });
  }, []);

  // ── Apply inline formatting to text block ─────────────────────────
  const applyFormat = useCallback(([before, after]) => {
    const blockId = fmtBlockId.current;
    if (!blockId) return;
    const block = blocks.find(b => b.id === blockId);
    if (!block) return;

    // Get selection in the contenteditable
    const sel = window.getSelection();
    const range = savedRange.current || (sel?.rangeCount > 0 ? sel.getRangeAt(0) : null);
    if (!range) return;

    const container = range.commonAncestorContainer;
    const el = container.nodeType === 3 ? container.parentElement : container;

    // Walk up to find the contenteditable
    let ceEl = el;
    while (ceEl && !ceEl.contentEditable?.toString() === 'true') ceEl = ceEl.parentElement;

    // Get plain text and selection offsets
    const fullText = block.text;
    const selText = range.toString();
    if (!selText) return;

    const start = fullText.indexOf(selText);
    if (start === -1) {
      // Fallback: wrap visible selected text
      const newText = fullText + before + selText + after;
      updateBlock({ ...block, text: newText });
      setFmtBar(null);
      return;
    }

    const newText = fullText.substring(0, start) + before + selText + after + fullText.substring(start + selText.length);
    updateBlock({ ...block, text: newText });
    setFmtBar(null);
  }, [blocks, updateBlock]);

  // ── Click outside → close menus, clear selection ─────────────────
  const handleEditorClick = useCallback((e) => {
    const clickedBlock = e.target.closest('[data-block-id]');
    if (!clickedBlock) {
      setFocusedId(null);
      setFmtBar(null);
      setElMenu(null);
    }
  }, []);

  // ── + button between blocks ────────────────────────────────────────
  const showInsertMenu = useCallback((e, afterId) => {
    e.stopPropagation();
    const rect = e.currentTarget.getBoundingClientRect();
    setElMenu({ x: rect.left + rect.width/2, y: rect.bottom + 8, afterId });
  }, []);

  // ── Word count ────────────────────────────────────────────────────
  const wordCount = useMemo(() => {
    const text = mode === 'write' ? rawMd : blocksToMd(blocks);
    return text.split(/\s+/).filter(Boolean).length;
  }, [blocks, rawMd, mode]);

  const charCount = useMemo(() => {
    return (mode === 'write' ? rawMd : blocksToMd(blocks)).length;
  }, [blocks, rawMd, mode]);

  // ── Loading ───────────────────────────────────────────────────────
  if (loading) return (
    <div className="flex items-center justify-center h-full bg-base-100">
      <span className="loading loading-spinner loading-md" />
    </div>
  );

  const MODES = [
    { id: 'rich',    icon: Type,    title: 'Rich — block editor' },
    { id: 'write',   icon: Edit3,   title: 'Markdown source' },
    { id: 'split',   icon: Columns, title: 'Split view' },
    { id: 'preview', icon: Eye,     title: 'Preview' },
  ];

  return (
    <div className="doc-editor-root flex flex-col h-full bg-base-100 relative" onClick={handleEditorClick}>

      {/* ── Top bar ── */}
      <div className="shrink-0 flex items-center justify-between px-4 py-2 border-b border-base-200 bg-base-100 z-10">
        <div className="flex items-center gap-3 min-w-0">
          <h2 className="text-sm font-semibold text-base-content truncate">{doc?.title}</h2>
          {dirty && <span className="text-xs text-base-content/40 shrink-0">Unsaved</span>}
        </div>
        <div className="flex items-center gap-1 shrink-0">
          <div className="flex items-center bg-base-200 rounded-lg p-0.5 mr-2">
            {MODES.map(m => (
              <button key={m.id} title={m.title} onClick={() => setMode(m.id)}
                className={`btn btn-xs btn-square ${mode===m.id ? 'bg-base-100 shadow-sm text-base-content' : 'btn-ghost text-base-content/40'}`}>
                <m.icon size={13}/>
              </button>
            ))}
          </div>
          <button className="btn btn-ghost btn-xs btn-square text-base-content/50" onClick={doRefresh} title="Refresh from GitHub">
            <RefreshCw size={14}/>
          </button>
          <button className="btn btn-ghost btn-xs btn-square text-base-content/50" onClick={doExport} title="Export .md">
            <Download size={14}/>
          </button>
          <button className={`btn btn-sm gap-1.5 ${dirty ? 'btn-primary' : 'btn-ghost'}`}
            onClick={doSave} disabled={saving || !dirty}>
            {saving ? <span className="loading loading-spinner loading-xs"/> : <Save size={14}/>}
            {saving ? 'Saving…' : 'Save'}
          </button>
          {onClose && (
            <button className="btn btn-ghost btn-xs btn-square ml-1" onClick={onClose} title="Close">
              <X size={14}/>
            </button>
          )}
        </div>
      </div>

      {/* ── Editor area ── */}
      <div ref={editorRef} className="flex-1 flex overflow-hidden min-h-0">

        {/* RICH BLOCK EDITOR */}
        {(mode === 'rich' || mode === 'split') && (
          <div className={`rich-panel flex flex-col overflow-auto ${mode==='split' ? 'w-1/2 border-r border-base-200' : 'flex-1'}`}>
            <div className="rich-inner">
              {blocks.map((block, idx) => (
                <div key={block.id} className="block-row">
                  <BlockWrapper
                    block={block}
                    focused={focusedId === block.id}
                    onFocus={setFocusedId}
                    onUpdate={updateBlock}
                    onDelete={() => deleteBlock(block.id)}
                    onCopy={() => copyBlock(block.id)}
                    onKeyDown={makeKeyDown(block)}
                    onSelectionChange={() => handleSelectionChange(block.id)}
                  />
                  <button className="insert-btn" onClick={e => showInsertMenu(e, block.id)} title="Insert block">
                    <Plus size={11}/>
                  </button>
                </div>
              ))}
              {/* Final add block button */}
              <div className="final-add-row">
                <button className="final-add-btn" onClick={e => {
                  const lastId = blocks[blocks.length-1]?.id;
                  const rect = e.currentTarget.getBoundingClientRect();
                  setElMenu({ x: rect.left + rect.width/2, y: rect.bottom + 8, afterId: lastId });
                }}>
                  <Plus size={13}/> Add block
                </button>
              </div>
            </div>
          </div>
        )}

        {/* MARKDOWN TEXTAREA */}
        {mode === 'write' && (
          <textarea ref={textareaRef}
            className="flex-1 w-full h-full resize-none outline-none bg-base-100 text-base-content text-sm leading-relaxed font-mono px-8 py-6 placeholder:text-base-content/25"
            placeholder="Write Markdown…"
            value={rawMd}
            onChange={e => { setRawMd(e.target.value); setDirty(true); }}
            spellCheck />
        )}

        {/* PREVIEW */}
        {(mode === 'preview' || mode === 'split') && (
          <div className={`preview-panel overflow-auto ${mode==='split' ? 'w-1/2' : 'flex-1'}`}>
            <div className="prose-wrap">
              {(mode === 'split' ? blocks : mdToBlocks(rawMd)).map(renderBlockPreview)}
            </div>
          </div>
        )}
      </div>

      {/* ── Status bar ── */}
      <div className="shrink-0 flex items-center justify-between px-4 py-1.5 border-t border-base-200 bg-base-200/40 text-xs text-base-content/40">
        <span>
          {mode === 'rich' ? `${blocks.length} blocks` : mode === 'write' ? 'Markdown' : mode === 'split' ? 'Split' : 'Preview'}
          {' · '}{wordCount} words · {charCount} chars
        </span>
        <span>
          {doc?.updatedAt
            ? `Saved ${new Date(doc.updatedAt).toLocaleString('en-US',{month:'short',day:'numeric',hour:'2-digit',minute:'2-digit'})}`
            : 'Not saved yet'}
        </span>
      </div>

      {/* ── Formatting bar ── */}
      {fmtBar && (
        <FormattingBar
          position={fmtBar}
          onFormat={applyFormat}
          onClose={() => setFmtBar(null)}
        />
      )}

      {/* ── Element insertion menu ── */}
      {elMenu && (
        <ElementMenu
          position={elMenu}
          onInsert={type => { insertBlockAfter(elMenu.afterId, type); setElMenu(null); }}
          onClose={() => setElMenu(null)}
        />
      )}

      {/* ═══════════════════ STYLES ═══════════════════ */}
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Lora:ital,wght@0,400;0,500;0,600;0,700;1,400&display=swap');

        .doc-editor-root, .doc-editor-root *:not(code):not(pre):not(textarea):not(.code-block-ta) {
          font-family: 'Lora', Georgia, serif;
        }
        .doc-editor-root textarea, .doc-editor-root code, .doc-editor-root pre,
        .doc-editor-root .code-block-ta { font-family: ui-monospace, 'Fira Code', Menlo, monospace !important; }

        /* ─── Rich panel ─────────────────────────────── */
        .rich-panel { background: oklch(var(--b1)); }
        .rich-inner { max-width: 720px; margin: 0 auto; padding: 2.5rem 1.5rem 6rem; }

        /* ─── Block row ──────────────────────────────── */
        .block-row { position: relative; margin-bottom: 2px; }
        .block-row:last-child { margin-bottom: 0; }

        /* ─── Insert button between blocks ───────────── */
        .insert-btn {
          position: absolute; left: -28px; top: 50%; transform: translateY(-50%);
          width: 20px; height: 20px; border-radius: 50%;
          background: oklch(var(--b2)/.7); border: 1px dashed oklch(var(--bc)/.2);
          color: oklch(var(--bc)/.35); cursor: pointer;
          display: flex; align-items: center; justify-content: center;
          opacity: 0; transition: opacity 0.15s;
          font-family: ui-sans-serif,system-ui,sans-serif;
        }
        .block-row:hover .insert-btn { opacity: 1; }
        .insert-btn:hover { background: oklch(var(--p)/.15); color: oklch(var(--p)); border-color: oklch(var(--p)/.4); opacity:1; }

        .final-add-row { margin-top: 1.5rem; display: flex; justify-content: center; }
        .final-add-btn {
          display: flex; align-items: center; gap: 6px;
          font-size: 12px; color: oklch(var(--bc)/.3); cursor: pointer;
          padding: 6px 14px; border-radius: 99px;
          border: 1px dashed oklch(var(--bc)/.15);
          background: transparent; transition: all 0.15s;
          font-family: ui-sans-serif,system-ui,sans-serif;
        }
        .final-add-btn:hover { color: oklch(var(--p)); border-color: oklch(var(--p)/.4); background: oklch(var(--p)/.06); }

        /* ─── Block wrapper & boundary box ───────────── */
        .block-wrap {
          position: relative; border-radius: 6px;
          border: 1px dashed transparent;
          transition: border-color 0.12s, background 0.12s;
          padding: 3px 6px;
        }
        /* Dashed boundary: alternating black/white dashes — visible in both themes */
        .block-wrap:hover, .block-wrap.focused {
          border-color: transparent;
          background-image: repeating-linear-gradient(0deg, oklch(var(--bc)/.3) 0, oklch(var(--bc)/.3) 4px, transparent 4px, transparent 8px),
                            repeating-linear-gradient(90deg, oklch(var(--bc)/.3) 0, oklch(var(--bc)/.3) 4px, transparent 4px, transparent 8px),
                            repeating-linear-gradient(180deg, oklch(var(--bc)/.3) 0, oklch(var(--bc)/.3) 4px, transparent 4px, transparent 8px),
                            repeating-linear-gradient(270deg, oklch(var(--bc)/.3) 0, oklch(var(--bc)/.3) 4px, transparent 4px, transparent 8px);
          background-size: 1px 100%, 100% 1px, 1px 100%, 100% 1px;
          background-position: 0 0, 0 0, 100% 0, 0 100%;
          background-repeat: no-repeat;
          background-color: oklch(var(--b2)/.35);
        }
        .block-wrap.focused {
          background-color: oklch(var(--b2)/.6);
        }
        /* Distinct focused style for void blocks */
        .block-wrap.void-block:hover, .block-wrap.void-block.focused {
          background-color: oklch(var(--p)/.04);
        }

        /* ─── Block overlay (copy/delete) ────────────── */
        .block-overlay {
          position: absolute; top: 4px; right: 4px;
          display: flex; gap: 3px; z-index: 20;
          opacity: 0; pointer-events: none; transition: opacity 0.12s;
          font-family: ui-sans-serif,system-ui,sans-serif;
        }
        .block-wrap:hover .block-overlay, .block-wrap.focused .block-overlay {
          opacity: 1; pointer-events: auto;
        }
        .bov-btn {
          display: flex; align-items: center; justify-content: center;
          width: 22px; height: 22px; border-radius: 5px; cursor: pointer;
          background: oklch(var(--b1)); border: 1px solid oklch(var(--bc)/.15);
          color: oklch(var(--bc)/.5); transition: all 0.1s;
        }
        .bov-btn:hover { background: oklch(var(--b3)); color: oklch(var(--bc)); }
        .bov-del:hover { background: oklch(var(--er)/.15); color: oklch(var(--er)); border-color: oklch(var(--er)/.3); }

        /* ─── Text block styles ──────────────────────── */
        .tblock {
          outline: none; width: 100%; min-height: 1.6em;
          word-break: break-word; white-space: pre-wrap;
        }
        .tblock:empty::before { content: attr(data-placeholder); color: oklch(var(--bc)/.25); pointer-events: none; }
        .tblock-paragraph  { font-size: .95rem; line-height: 1.9; padding: .25rem 0; }
        .tblock-h1 { font-size: 1.85rem; font-weight: 700; line-height: 1.2; padding: .6rem 0 .3rem; border-bottom: 1px solid oklch(var(--bc)/.1); margin-bottom: .3rem; }
        .tblock-h2 { font-size: 1.4rem; font-weight: 600; line-height: 1.25; padding: .5rem 0 .25rem; border-bottom: 1px solid oklch(var(--bc)/.07); }
        .tblock-h3 { font-size: 1.15rem; font-weight: 600; line-height: 1.3; padding: .4rem 0; }
        .tblock-h4 { font-size: 1rem; font-weight: 600; line-height: 1.35; padding: .35rem 0; }
        .tblock-blockquote {
          border-left: 4px solid oklch(var(--p)/.45); padding: .5rem 1rem;
          color: oklch(var(--bc)/.6); font-style: italic; margin: 0;
          background: oklch(var(--b2)/.5); border-radius: 0 .5rem .5rem 0;
        }
        /* Inline formatting inside text blocks */
        .tblock strong { font-weight: 700; }
        .tblock em     { font-style: italic; }
        .tblock s      { text-decoration: line-through; color: oklch(var(--bc)/.5); }
        .tblock mark   { background: #fef08a; color: #111; border-radius: .2em; padding: 0 .1em; }
        .tblock code   { font-size: .82em; background: oklch(var(--b2)); padding: .1em .35em; border-radius: .3rem; }
        .tblock sup, .tblock sub { font-size: .75em; }
        .tblock a      { color: oklch(var(--p)); text-decoration: underline; }

        /* ─── Code block ─────────────────────────────── */
        .code-block-editor { border-radius: 8px; overflow: hidden; border: 1px solid oklch(var(--bc)/.12); background: oklch(var(--b2)); }
        .code-block-header {
          display: flex; align-items: center; gap: 8px;
          padding: 6px 12px; border-bottom: 1px solid oklch(var(--bc)/.1);
          background: oklch(var(--b3)/.5);
          font-family: ui-sans-serif,system-ui,sans-serif;
        }
        .code-lang-sel {
          font-size: 11px; background: transparent; border: none; outline: none;
          color: oklch(var(--bc)/.6); cursor: pointer;
          font-family: ui-monospace, Menlo, monospace;
        }
        .code-block-ta {
          width: 100%; background: transparent; border: none; outline: none; resize: none;
          padding: 1rem 1.2rem; font-size: .82rem; line-height: 1.65; color: oklch(var(--bc)/.85);
          tab-size: 2;
        }

        /* ─── List editor ────────────────────────────── */
        .list-editor { padding: .3rem 0; }
        .list-item-row { display: flex; align-items: center; gap: 7px; padding: 3px 0; }
        .task-cb { accent-color: oklch(var(--p)); flex-shrink: 0; }
        .ol-num { font-size: .9rem; color: oklch(var(--bc)/.5); min-width: 1.6rem; text-align: right; flex-shrink:0; font-family: ui-monospace, monospace; }
        .ul-dot { font-size: 1.2rem; color: oklch(var(--bc)/.5); line-height: 1; flex-shrink:0; }
        .list-item-input {
          flex: 1; background: transparent; border: none; outline: none;
          font-size: .93rem; color: oklch(var(--bc)); line-height: 1.6;
          font-family: 'Lora', Georgia, serif;
        }
        .list-item-input.done { text-decoration: line-through; color: oklch(var(--bc)/.45); }
        .list-item-input::placeholder { color: oklch(var(--bc)/.25); }
        .list-del-btn { opacity: 0; padding: 2px; color: oklch(var(--bc)/.35); transition: opacity 0.1s; cursor: pointer; flex-shrink:0; }
        .list-item-row:hover .list-del-btn { opacity: 1; }
        .list-del-btn:hover { color: oklch(var(--er)); }
        .list-add-btn {
          display: flex; align-items: center; gap: 5px; margin-top: 6px;
          font-size: 12px; color: oklch(var(--bc)/.35); cursor: pointer;
          padding: 4px 8px; border-radius: 5px; background: transparent;
          border: 1px dashed oklch(var(--bc)/.15); transition: all 0.12s;
          font-family: ui-sans-serif,system-ui,sans-serif;
        }
        .list-add-btn:hover { color: oklch(var(--p)); border-color: oklch(var(--p)/.3); background: oklch(var(--p)/.06); }

        /* ─── Table editor ───────────────────────────── */
        .table-editor { overflow: visible; }
        .table-wrap { overflow-x: auto; }
        .te-table { border-collapse: collapse; width: 100%; min-width: 300px; font-size: .88rem; }
        .te-cell { padding: 0; border: 1px solid oklch(var(--bc)/.13); position: relative; }
        .te-hcell { background: oklch(var(--b2)/.7); font-weight: 600; }
        .te-input {
          width: 100%; padding: 6px 10px; background: transparent;
          border: none; outline: none; font-size: .88rem;
          font-family: 'Lora', Georgia, serif; color: oklch(var(--bc));
        }
        .te-del-col {
          position: absolute; top: -8px; right: -8px; width: 16px; height: 16px;
          border-radius: 50%; background: oklch(var(--er)/.15); color: oklch(var(--er));
          display: none; align-items: center; justify-content: center; cursor: pointer; z-index:5; border:none;
        }
        .te-hcell:hover .te-del-col { display: flex; }
        .te-row-ctrl { border: none; background: transparent; padding: 0 2px; width: 20px; }
        .te-row-ctrl button { opacity:0; color: oklch(var(--er)); cursor:pointer; background:none; border:none; padding:2px; }
        .te-table tr:hover .te-row-ctrl button { opacity:1; }
        .table-btns { display: flex; gap: 8px; margin-top: 8px; font-family: ui-sans-serif,system-ui,sans-serif; }
        .te-add-btn {
          display: flex; align-items: center; gap: 4px; font-size: 11px;
          color: oklch(var(--bc)/.4); padding: 4px 10px; border-radius: 5px;
          border: 1px dashed oklch(var(--bc)/.15); background: transparent; cursor: pointer;
        }
        .te-add-btn:hover { color: oklch(var(--p)); border-color: oklch(var(--p)/.3); background: oklch(var(--p)/.06); }

        /* ─── Image editor ───────────────────────────── */
        .img-editor { padding: .25rem 0; }
        .img-fields { display: flex; flex-direction: column; gap: 6px; margin-bottom: 8px; }
        .img-input {
          width: 100%; padding: 7px 12px; border-radius: 6px;
          border: 1px solid oklch(var(--bc)/.15); background: oklch(var(--b2)/.5);
          font-size: .85rem; outline: none; color: oklch(var(--bc));
          font-family: ui-monospace, Menlo, monospace;
        }
        .img-alt { font-family: 'Lora', Georgia, serif !important; }
        .img-input:focus { border-color: oklch(var(--p)/.5); }
        .img-preview-wrap { margin-top: 8px; border-radius: 8px; overflow: hidden; }
        .img-preview { max-width: 100%; max-height: 280px; object-fit: contain; border-radius: 6px; display: block; }

        /* ─── Link editor ────────────────────────────── */
        .link-editor {
          display: flex; align-items: center; gap: 10px;
          padding: .4rem .5rem; border-radius: 8px;
          background: oklch(var(--b2)/.5); border: 1px solid oklch(var(--bc)/.1);
        }
        .link-icon-wrap { color: oklch(var(--p)); flex-shrink: 0; }
        .link-fields { flex: 1; display: flex; flex-direction: column; gap: 4px; }
        .link-input {
          border: none; outline: none; background: transparent;
          font-size: .88rem; color: oklch(var(--bc));
        }
        .link-text { font-weight: 500; font-family: 'Lora', Georgia, serif; }
        .link-url { font-size: .8rem; color: oklch(var(--bc)/.5); font-family: ui-monospace, Menlo, monospace !important; }
        .link-url::placeholder { color: oklch(var(--bc)/.25); }
        .link-preview {
          display: flex; align-items: center; gap: 4px; font-size: .78rem;
          color: oklch(var(--p)); text-decoration: none; white-space: nowrap;
          font-family: ui-sans-serif,system-ui,sans-serif;
        }

        /* ─── Alert editor ───────────────────────────── */
        .alert-editor { padding: .25rem 0; }
        .alert-type-row { display: flex; flex-wrap: wrap; gap: 5px; margin-bottom: 8px; font-family: ui-sans-serif,system-ui,sans-serif; }
        .alert-type-btn {
          display: flex; align-items: center; gap: 5px; font-size: 11px;
          padding: 4px 9px; border-radius: 99px; cursor: pointer;
          border: 1px solid oklch(var(--bc)/.15); background: transparent;
          color: oklch(var(--bc)/.5); transition: all 0.12s;
        }
        .alert-type-btn.active, .alert-type-btn:hover { border-color: transparent; }
        .alert-type-btn[data-type="NOTE"].active    { background: #dbeafe; color: #1d4ed8; }
        .alert-type-btn[data-type="TIP"].active     { background: #dcfce7; color: #15803d; }
        .alert-type-btn[data-type="WARNING"].active { background: #fef9c3; color: #a16207; }
        .alert-type-btn[data-type="IMPORTANT"].active { background: #ede9fe; color: #7c3aed; }
        .alert-type-btn[data-type="CAUTION"].active { background: #fee2e2; color: #b91c1c; }
        .alert-body { padding: .75rem 1rem; border-left: 4px solid; border-radius: 0 .5rem .5rem 0; }
        .alert-label-row { display: flex; align-items: center; gap: 6px; font-weight: 700; font-size: .75rem; letter-spacing: .07em; text-transform: uppercase; margin-bottom: .4rem; font-family: ui-sans-serif,system-ui,sans-serif; }
        .alert-ta {
          width: 100%; background: transparent; border: none; outline: none; resize: none;
          font-size: .9rem; line-height: 1.7; font-family: 'Lora', Georgia, serif; color: inherit;
        }

        /* ─── HR block ───────────────────────────────── */
        .hr-editor { padding: .6rem 0; }
        .hr-visual { border: none; border-top: 2px solid oklch(var(--bc)/.15); margin: .3rem 0; }

        /* ─── Formatting bar ─────────────────────────── */
        .fmt-bar {
          position: fixed; z-index: 300;
          display: flex; align-items: center; gap: 1px;
          background: oklch(var(--b1)); border: 1px solid oklch(var(--bc)/.15);
          border-radius: 10px; padding: 4px 6px;
          box-shadow: 0 4px 20px rgba(0,0,0,.15), 0 1px 4px rgba(0,0,0,.08);
          font-family: ui-sans-serif,system-ui,sans-serif;
        }
        .fmt-btn {
          display: flex; align-items: center; justify-content: center;
          width: 26px; height: 26px; border-radius: 6px; cursor: pointer;
          background: transparent; color: oklch(var(--bc)/.6); border: none;
          transition: background 0.1s, color 0.1s;
        }
        .fmt-btn:hover { background: oklch(var(--b2)); color: oklch(var(--bc)); }
        .fmt-sep { width: 1px; height: 16px; background: oklch(var(--bc)/.15); margin: 0 2px; }
        .fmt-close:hover { background: oklch(var(--er)/.1); color: oklch(var(--er)); }

        /* ─── Element insertion menu ─────────────────── */
        .el-menu {
          position: fixed; z-index: 300; width: 210px;
          background: oklch(var(--b1)); border: 1px solid oklch(var(--bc)/.12);
          border-radius: 12px; padding: 6px;
          box-shadow: 0 8px 32px rgba(0,0,0,.18), 0 2px 8px rgba(0,0,0,.1);
          font-family: ui-sans-serif,system-ui,sans-serif;
          max-height: 400px; overflow-y: auto;
        }
        .el-menu-header {
          display: flex; align-items: center; justify-content: space-between;
          padding: 4px 6px 6px; border-bottom: 1px solid oklch(var(--bc)/.08); margin-bottom: 4px;
        }
        .el-menu-header span { font-size: 11px; font-weight: 600; color: oklch(var(--bc)/.5); text-transform: uppercase; letter-spacing: .06em; }
        .el-close { background: none; border: none; cursor: pointer; color: oklch(var(--bc)/.4); padding: 2px; display:flex; align-items:center; }
        .el-group-label { font-size: 10px; font-weight: 700; color: oklch(var(--bc)/.35); letter-spacing: .08em; text-transform: uppercase; padding: 6px 8px 3px; }
        .el-item {
          display: flex; align-items: center; gap: 9px; width: 100%;
          padding: 7px 8px; border-radius: 7px; cursor: pointer;
          background: none; border: none; font-size: 13px; color: oklch(var(--bc)/.8);
          text-align: left; transition: background 0.1s;
        }
        .el-item:hover { background: oklch(var(--b2)); color: oklch(var(--bc)); }
        .el-item-icon { color: oklch(var(--bc)/.4); flex-shrink: 0; }

        /* ─── Preview panel ──────────────────────────── */
        .preview-panel { background: oklch(var(--b1)); }
        .prose-wrap { max-width: 720px; margin: 0 auto; padding: 2.5rem 2rem 6rem; }
        .prose-wrap p  { margin: .6rem 0; line-height: 1.9; font-size: .95rem; }
        .prose-wrap h1 { font-size: 1.85rem; font-weight: 700; line-height: 1.2; border-bottom: 1px solid oklch(var(--bc)/.1); padding-bottom: .3rem; margin: 1.5rem 0 .6rem; }
        .prose-wrap h2 { font-size: 1.4rem; font-weight: 600; border-bottom: 1px solid oklch(var(--bc)/.07); padding-bottom: .2rem; margin: 1.3rem 0 .5rem; }
        .prose-wrap h3 { font-size: 1.15rem; font-weight: 600; margin: 1.1rem 0 .4rem; }
        .prose-wrap h4 { font-size: 1rem; font-weight: 600; margin: .9rem 0 .35rem; }
        .prose-wrap blockquote { border-left: 4px solid oklch(var(--p)/.4); padding: .5rem 1rem; color: oklch(var(--bc)/.65); font-style: italic; margin: .8rem 0; background: oklch(var(--b2)/.5); border-radius: 0 .5rem .5rem 0; }
        .prose-wrap hr { border: none; border-top: 2px solid oklch(var(--bc)/.12); margin: 1.5rem 0; }
        .prose-wrap ul, .prose-wrap ol { padding-left: 1.6rem; margin: .5rem 0; }
        .prose-wrap li { margin: .3rem 0; line-height: 1.8; }
        .prose-wrap ul.task-list { list-style: none; padding-left: .4rem; }
        .prose-wrap .task-list li { display: flex; align-items: flex-start; gap: .5rem; }
        .prose-wrap .task-list input { margin-top: .25rem; accent-color: oklch(var(--p)); }
        .prose-wrap .task-done span { text-decoration: line-through; color: oklch(var(--bc)/.45); }
        .prose-wrap code { font-size: .82em; background: oklch(var(--b2)); padding: .12em .38em; border-radius: .3rem; }
        .prose-wrap a { color: oklch(var(--p)); text-decoration: underline; }
        .prose-wrap strong { font-weight: 700; }
        .prose-wrap em { font-style: italic; }
        .prose-wrap s { text-decoration: line-through; color: oklch(var(--bc)/.5); }
        .prose-wrap mark { background: #fef08a; color: #111; border-radius: .2em; padding: 0 .1em; }
        .prose-wrap sup, .prose-wrap sub { font-size: .75em; }
        .prose-wrap table { border-collapse: collapse; width: 100%; margin: .8rem 0; font-size: .9em; }
        .prose-wrap th { background: oklch(var(--b2)); font-weight: 600; padding: .5rem .75rem; border: 1px solid oklch(var(--bc)/.15); text-align: left; }
        .prose-wrap td { padding: .45rem .75rem; border: 1px solid oklch(var(--bc)/.12); }
        .prose-wrap tr:nth-child(even) td { background: oklch(var(--b2)/.4); }
        .prose-wrap img { max-width: 100%; border-radius: .6rem; }
        .prose-wrap figure { margin: 1rem 0; }
        .prose-wrap figcaption { font-size: .8rem; color: oklch(var(--bc)/.5); text-align: center; margin-top: .4rem; }
        .preview-code {
          background: oklch(var(--b2)); border-radius: .6rem; padding: 1rem 1.2rem;
          overflow-x: auto; margin: .9rem 0; border: 1px solid oklch(var(--bc)/.08);
          position: relative;
        }
        .preview-code code { font-size: .82em; background: none; padding: 0; line-height: 1.65; }
        .code-lang-badge {
          position: absolute; top: 8px; right: 10px;
          font-size: 10px; color: oklch(var(--bc)/.4);
          font-family: ui-monospace, Menlo, monospace;
        }
        .preview-link-block a {
          display: inline-flex; align-items: center; gap: 5px;
          padding: 6px 14px; border-radius: 8px;
          background: oklch(var(--b2)); border: 1px solid oklch(var(--bc)/.12);
          color: oklch(var(--p)); text-decoration: none; font-size: .9rem;
        }
        /* Preview alerts */
        .preview-alert { border-radius: .5rem; padding: .75rem 1rem; margin: .8rem 0; border-left: 4px solid; }
        .preview-alert-header { display: flex; align-items: center; gap: 6px; font-weight: 700; font-size: .75rem; letter-spacing: .07em; text-transform: uppercase; margin-bottom: .4rem; font-family: ui-sans-serif,system-ui,sans-serif; }
        .preview-alert-note      { background: #eff6ff; border-color: #3b82f6; color: #1d4ed8; }
        .preview-alert-tip       { background: #f0fdf4; border-color: #10b981; color: #065f46; }
        .preview-alert-warning   { background: #fffbeb; border-color: #f59e0b; color: #92400e; }
        .preview-alert-important { background: #f5f3ff; border-color: #8b5cf6; color: #4c1d95; }
        .preview-alert-caution   { background: #fef2f2; border-color: #ef4444; color: #991b1b; }
        .preview-alert div { color: oklch(var(--bc)); font-size: .9rem; line-height: 1.7; }
      `}</style>
    </div>
  );
};

export default DocEditor;
```