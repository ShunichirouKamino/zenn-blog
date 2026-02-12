# Svelte 5 は AI コーディング時代の最適解か？ ― React との比較から見えてくる開発体験の違い

## はじめに

ChatGPT や Claude Code をはじめとする AI コーディングツールの普及により、フレームワーク選定の基準が変わりつつあります。「人間が書きやすいか」だけでなく、**「AI が正確に生成・理解しやすいか」** という観点が重要度を増しています。

本記事では、Svelte 5 のサンプルアプリ（タスク管理アプリ）を題材に、React との具体的なコード比較を通じて、AI 開発における Svelte の優位性を検証します。

---

## 1. Svelte の概要

### Svelte とは

**Svelte**（スベルト）は、2016 年 11 月に **Rich Harris** 氏（当時 The Guardian 所属、現 Vercel）によって公開されたフロントエンドフレームワークです。React や Vue がブラウザ上で仮想 DOM を用いてレンダリングするのに対し、Svelte は**ビルド時にコンパイル**を行い、仮想 DOM を使わず最小限の JavaScript を出力するという根本的に異なるアプローチを取ります。

### バージョンの変遷

| バージョン | リリース年 | 主な特徴 |
|-----------|-----------|---------|
| Svelte 1 | 2016年 | 初版リリース。コンパイラベースのアプローチ |
| Svelte 2 | 2018年 | テンプレート構文の改良 |
| Svelte 3 | 2019年 | リアクティビティの大幅刷新。`$:` ラベルによるリアクティブ宣言 |
| Svelte 4 | 2023年 | パフォーマンス改善、TypeScript サポート強化 |
| **Svelte 5** | **2024年10月** | **Runes（ルーン）導入。シグナルベースのリアクティビティ** |

**現在の最新バージョンは Svelte 5 系（5.x）** であり、SvelteKit と合わせてフルスタックな Web 開発が可能です。

### Svelte 5 の主要な特徴

Svelte 5 では **Runes** と呼ばれる新しいリアクティビティシステムが導入されました。`$` プレフィックス付きの関数（`$state`、`$derived`、`$effect`、`$props` など）でリアクティビティを宣言します。

以下は、サンプルアプリのカウンターコンポーネントです。

**Svelte 5 — `Counter.svelte`**

```svelte
<script lang="ts">
  let count = $state(0);
  let doubled = $derived(count * 2);
  let quadrupled = $derived(doubled * 2);
</script>

<p>{count}</p>
<p>x2 = {doubled} / x4 = {quadrupled}</p>
<button onclick={() => count--}>-1</button>
<button onclick={() => count++}>+1</button>
<button onclick={() => count = 0}>リセット</button>
```

**React での同等コード**

```tsx
import { useState, useMemo } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  const doubled = useMemo(() => count * 2, [count]);
  const quadrupled = useMemo(() => doubled * 2, [doubled]);

  return (
    <>
      <p>{count}</p>
      <p>x2 = {doubled} / x4 = {quadrupled}</p>
      <button onClick={() => setCount(c => c - 1)}>-1</button>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
      <button onClick={() => setCount(0)}>リセット</button>
    </>
  );
}
```

Svelte 5 では `let count = $state(0)` の 1 行で済むリアクティブ変数宣言が、React では `useState` のインポート、分割代入、更新関数の呼び出しという 3 段階を踏む必要があります。`$derived` も依存配列なしで自動追跡されるため、`useMemo` の依存配列忘れによるバグが原理的に発生しません。

---

## 2. React にはあって Svelte にはない機能

### 2-1. 仮想 DOM（Virtual DOM）とその周辺エコシステム

React の根幹である仮想 DOM は、Svelte には**意図的に存在しません**。Svelte はコンパイラが最適な DOM 操作コードを直接生成するため、仮想 DOM の差分比較（diffing）自体が不要です。

これは AI 開発の観点からも意味があります。仮想 DOM の振る舞いを踏まえた `React.memo`、`useCallback`、`useMemo` といった最適化コードを AI が生成する必要がなく、パフォーマンスチューニングのミスが起きにくくなります。

### 2-2. useCallback / useMemo（メモ化 API）

React では再レンダリングを抑制するために `useCallback` や `useMemo` を使いこなす必要があります。

```tsx
// React: 依存配列の管理が必要
const handleToggle = useCallback((id: number) => {
  setTodos(prev => prev.map(t => t.id === id ? { ...t, done: !t.done } : t));
}, []);

const stats = useMemo(() => ({
  total: todos.length,
  done: todos.filter(t => t.done).length,
}), [todos]);
```

Svelte 5 のサンプルアプリでは、同等の算出値が `$derived` で簡潔に書けます。

**Svelte 5 — `todo.svelte.ts`（共有ステート）**

```ts
let todos = $state<Todo[]>([...]);

const stats = $derived({
  total: todos.length,
  done: todos.filter((t) => t.done).length,
  active: todos.filter((t) => !t.done).length,
  highPriority: todos.filter((t) => t.priority === 'high' && !t.done).length,
});

export function toggleTodo(id: number) {
  const todo = todos.find((t) => t.id === id);
  if (todo) todo.done = !todo.done;
}
```

`$derived` は依存関係を自動追跡するため、依存配列の指定が不要です。関数のメモ化（`useCallback`）に相当する仕組みもそもそも必要ありません。**AI にとって「依存配列に何を入れるべきか」を判断する必要がない**ことは、生成コードの正確性を大幅に向上させます。

### 2-3. JSX / Fragment

React では UI を JSX（JavaScript XML）で記述し、複数要素を返すときは `<Fragment>` や `<>...</>` でラップします。

```tsx
// React: JSX + Fragment が必要
return (
  <>
    <h1>タスク管理</h1>
    <TodoForm />
    <ul>
      {todos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  </>
);
```

Svelte ではテンプレート構文を使い、Fragment は不要です。

**Svelte 5 — `+page.svelte`**

```svelte
<h1>Svelte 5 タスク管理</h1>
<TodoForm />
<ul>
  {#each getTodos() as todo (todo.id)}
    <TodoItem {todo} />
  {/each}
</ul>
```

Svelte のテンプレート構文は HTML にきわめて近く、Fragment の概念がなく、`{#each}` ブロックでループを表現します。AI が HTML の知識をそのまま活用でき、JSX 特有の落とし穴（`className` vs `class`、`htmlFor` vs `for` など）を回避できます。

---

## 3. Svelte にはあって React にはない機能

### 3-1. 組み込みトランジション / アニメーション

Svelte はトランジション・アニメーションがフレームワーク組み込みです。React ではサードパーティライブラリ（Framer Motion、react-spring 等）が必要です。

**Svelte 5 — `TodoItem.svelte`**

```svelte
<script lang="ts">
  import { slide } from 'svelte/transition';
  import { toggleTodo, removeTodo } from '$lib/stores/todo.svelte';

  let { todo }: { todo: Todo } = $props();
</script>

<li class="todo-item" class:done={todo.done} transition:slide={{ duration: 200 }}>
  <label>
    <input type="checkbox" checked={todo.done} onchange={() => toggleTodo(todo.id)} />
    <span>{todo.text}</span>
  </label>
  <button onclick={() => removeTodo(todo.id)}>削除</button>
</li>
```

**React での同等コード（Framer Motion 使用）**

```tsx
import { AnimatePresence, motion } from 'framer-motion';

function TodoItem({ todo }: { todo: Todo }) {
  return (
    <motion.li
      initial={{ height: 0, opacity: 0 }}
      animate={{ height: 'auto', opacity: 1 }}
      exit={{ height: 0, opacity: 0 }}
      transition={{ duration: 0.2 }}
      className={`todo-item ${todo.done ? 'done' : ''}`}
    >
      <label>
        <input type="checkbox" checked={todo.done} onChange={() => toggleTodo(todo.id)} />
        <span>{todo.text}</span>
      </label>
      <button onClick={() => removeTodo(todo.id)}>削除</button>
    </motion.li>
  );
}
```

Svelte では `transition:slide` の一言で済む処理が、React では外部ライブラリのインポート、`AnimatePresence` による囲み、`motion.li` への置換、初期/終了/アニメーション状態の個別指定が必要です。AI が生成するコードとしても、Svelte の方が圧倒的にシンプルで、バグの混入リスクが低くなります。

### 3-2. 双方向バインディング（`bind:` ディレクティブ）

Svelte は `bind:value` でフォーム要素と変数を双方向に結びつけます。React には双方向バインディングの仕組みがなく、`value` + `onChange` の組み合わせが常に必要です。

**Svelte 5 — `TodoForm.svelte`**

```svelte
<script lang="ts">
  let newText = $state('');
  let priority = $state<Todo['priority']>('medium');

  function handleSubmit(e: Event) {
    e.preventDefault();
    if (newText.trim()) {
      addTodo(newText.trim(), priority);
      newText = '';
      priority = 'medium';
    }
  }
</script>

<form onsubmit={handleSubmit}>
  <input type="text" bind:value={newText} placeholder="新しいタスクを入力..." />
  <select bind:value={priority}>
    <option value="high">高</option>
    <option value="medium">中</option>
    <option value="low">低</option>
  </select>
  <button type="submit" disabled={!newText.trim()}>追加</button>
</form>
```

**React での同等コード**

```tsx
function TodoForm() {
  const [newText, setNewText] = useState('');
  const [priority, setPriority] = useState<Priority>('medium');

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    if (newText.trim()) {
      addTodo(newText.trim(), priority);
      setNewText('');
      setPriority('medium');
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={newText}
        onChange={(e) => setNewText(e.target.value)}
        placeholder="新しいタスクを入力..."
      />
      <select value={priority} onChange={(e) => setPriority(e.target.value as Priority)}>
        <option value="high">高</option>
        <option value="medium">中</option>
        <option value="low">低</option>
      </select>
      <button type="submit" disabled={!newText.trim()}>追加</button>
    </form>
  );
}
```

`bind:value={newText}` の 1 ディレクティブ vs `value={newText} onChange={(e) => setNewText(e.target.value)}` の 2 属性。フォームの入力フィールドが増えるほど、この差は広がります。AI がフォームを生成する際のコード量とエラー可能性が大きく異なります。

### 3-3. スコープ付き CSS と `class:` ディレクティブ

Svelte は `<style>` ブロックがデフォルトでコンポーネントスコープとなり、CSS Modules や styled-components のような追加ツールは不要です。さらに `class:done={todo.done}` のように条件付きクラスを簡潔に適用できます。

**Svelte 5 — `TodoItem.svelte`（スタイル部分）**

```svelte
<!-- class: ディレクティブで条件付きクラスを適用 -->
<li class="todo-item" class:done={todo.done}>
  ...
</li>

<style>
  .todo-item {
    display: flex;
    align-items: center;
    padding: 0.75rem 1rem;
    border-radius: 8px;
    background: var(--surface);
    border: 1px solid var(--border);
    transition: opacity 0.2s;
  }
  .todo-item.done { opacity: 0.5; }
  .todo-item.done .todo-text { text-decoration: line-through; }
</style>
```

**React での同等コード**

```tsx
// CSS Modules をインポート
import styles from './TodoItem.module.css';

// className の条件分岐を自前で管理
<li className={`${styles.todoItem} ${todo.done ? styles.done : ''}`}>
  ...
</li>
```

Svelte では `class:done={todo.done}` と書くだけで条件付きクラスが適用され、`<style>` タグ内の CSS は自動的にスコープ化されます。React では CSS Modules や clsx ライブラリの導入、テンプレートリテラルでのクラス名結合が必要です。

---

## 4. Svelte 5 の追加の注目機能

### 4-1. Snippets（再利用可能なテンプレートブロック）

Svelte 5 で導入された Snippets は、コンポーネント内で再利用可能なマークアップブロックを定義する機能です。

**Svelte 5 — `StatsPanel.svelte`**

```svelte
<!-- Snippet の定義 -->
{#snippet statCard(label: string, value: number, color: string)}
  <div class="stat-card" style="--accent: {color}">
    <span class="stat-value">{value}</span>
    <span class="stat-label">{label}</span>
  </div>
{/snippet}

<div class="stats-grid">
  <!-- @render で呼び出し -->
  {@render statCard('全タスク', getStats().total, '#6366f1')}
  {@render statCard('完了', getStats().done, '#22c55e')}
  {@render statCard('未完了', getStats().active, '#f59e0b')}
  {@render statCard('高優先度', getStats().highPriority, '#ef4444')}
</div>
```

React ではコンポーネントを分離するか、レンダー関数を定義して同等のことを実現しますが、Svelte の Snippets はテンプレート内で完結するためコード量が少なくなります。

### 4-2. ユニバーサルリアクティビティ（`.svelte.ts` ファイル）

Svelte 5 では `.svelte.ts` / `.svelte.js` ファイルでコンポーネント外にリアクティブな状態を定義できます。

**Svelte 5 — `todo.svelte.ts`**

```ts
// コンポーネント外でも $state, $derived が使える
let todos = $state<Todo[]>([...]);
let filter = $state<'all' | 'active' | 'done'>('all');

const filteredTodos = $derived(
  filter === 'all'
    ? todos
    : filter === 'active'
      ? todos.filter((t) => !t.done)
      : todos.filter((t) => t.done)
);

export function getTodos() {
  return filteredTodos;
}

export function addTodo(text: string, priority: Todo['priority'] = 'medium') {
  todos.push({ id: nextId++, text, done: false, priority, createdAt: new Date() });
}
```

React では、コンポーネント外の共有ステートには Context API + useReducer、あるいは Zustand / Jotai / Redux といった外部状態管理ライブラリが必要です。Svelte 5 では追加ライブラリなしに、`.svelte.ts` ファイル内で `$state` と `$derived` をそのまま使えます。

### 4-3. `$effect` による副作用管理

**Svelte 5 — `theme.svelte.ts`**

```ts
let dark = $state(false);

$effect.root(() => {
  $effect(() => {
    if (typeof document !== 'undefined') {
      document.documentElement.classList.toggle('dark', dark);
      localStorage.setItem('theme', dark ? 'dark' : 'light');
    }
  });
});
```

React の `useEffect` では依存配列の指定が必須ですが、Svelte 5 の `$effect` は使用するリアクティブ変数を自動追跡します。

---

## 5. AI 開発との相性 ― 比較表

| 観点 | Svelte 5 | React |
|------|----------|-------|
| **コード量（ボイラープレート）** | 少ない。`$state(0)` で完結 | 多い。`useState` + setter + import |
| **依存配列の管理** | 不要。`$derived`、`$effect` は自動追跡 | 必要。`useMemo`、`useCallback`、`useEffect` すべてに依存配列が必須 |
| **AI が依存配列を間違えるリスク** | なし | 高い。無限ループや stale closure の原因となる |
| **フォーム処理** | `bind:value` で 1 行 | `value` + `onChange` で 2 行以上 |
| **CSS スコーピング** | 組み込み | CSS Modules / styled-components 等が必要 |
| **アニメーション** | `transition:slide` など組み込み | Framer Motion 等の外部ライブラリが必要 |
| **状態管理（コンポーネント外）** | `.svelte.ts` で `$state` をそのまま使用 | Context API / Redux / Zustand 等が必要 |
| **HTML との近さ** | `class`、`for` など標準属性をそのまま使用 | `className`、`htmlFor` など JSX 独自の変換が必要 |
| **学習コスト（AI含む）** | 低い。HTML + 少数の Runes | 高い。Hooks のルール、JSX の制約、メモ化戦略 |
| **コンパイルエラーの検出** | コンパイラが多くのミスを事前に検出 | ランタイムエラーが多い（Hooks のルール違反など） |
| **生成コードの正確性（AI）** | 高い。ルールが少なく間違えにくい | 中程度。Hooks のルールや依存配列で間違いやすい |
| **エコシステムの規模** | 中規模。成長中 | 大規模。最も豊富なサードパーティライブラリ |
| **AI の学習データ量** | 中程度（特に Svelte 5 Runes は新しい） | 非常に多い。Stack Overflow、GitHub に膨大な情報 |
| **1 プロンプトでの生成成功率** | 高い。構文がシンプルで一発で正しいコードになりやすい | 中程度。Hooks のルールに抵触するコードを生成しがち |

---

## 6. 総括

### Svelte 5 が AI コーディングに適している理由

**1. 「間違えにくい設計」が AI 生成の品質を底上げする**

AI コーディングにおいて最も問題になるのは、**文法的には正しいが意味的に間違ったコード**の生成です。React の依存配列は典型例で、`useMemo(() => compute(a, b), [a])` のように `b` を依存配列に入れ忘れても構文エラーにはなりません。Svelte 5 の `$derived` は依存配列という概念自体がないため、こうした「微妙なバグ」が原理的に発生しません。

**2. ボイラープレートの少なさがトークンを節約する**

AI は入出力のトークン数に制約があります。同じ機能を実現するのに Svelte 5 は React よりも 30〜50% 少ないコード量で済むケースが多く、限られたトークン内でより多くの機能を生成できます。

**3. HTML に近い構文が AI の既存知識を活かす**

AI の学習データには膨大な HTML が含まれています。Svelte のテンプレート構文は HTML にきわめて近いため、AI はその知識をそのまま活用できます。JSX 特有の変換（`class` → `className` など）によるミスが起きません。

### React が依然として有利な場面

一方で、React には以下の強みがあります。

- **エコシステムの規模**: npm パッケージ、UI ライブラリ（MUI、shadcn/ui など）の数で圧倒的
- **AI の学習データ量**: React に関する情報が圧倒的に多く、AI が複雑なパターンにも対応しやすい
- **大規模チーム開発の実績**: エンタープライズでの採用事例が豊富
- **React Server Components**: サーバーサイドの最新アーキテクチャ

### 結論

**AI コーディングとの相性という観点では、Svelte 5 は React よりも有利な特性を多く持っています。** 特にコード量の少なさ、依存配列が不要な設計、HTML に近いテンプレート構文は、AI が「正しいコードを一発で生成する」確率を高めます。

ただし、これは「Svelte が React より優れている」という単純な結論ではありません。React の巨大なエコシステムと AI 学習データの豊富さは現時点で大きなアドバンテージです。

現実的な指針としては:

- **新規の小〜中規模プロジェクト**で AI コーディングを積極活用するなら → **Svelte 5 を推奨**
- **大規模チーム開発**やエンタープライズ案件で豊富なライブラリ選択肢が必要なら → **React を推奨**
- **AI ツールの進化**により、Svelte 5 の学習データ不足は急速に解消されつつある

AI コーディングの時代において、「シンプルさ」は単なる好みの問題ではなく、**生成コードの品質に直結する技術的な利点**です。Svelte 5 の設計哲学は、この時代に合致していると言えるでしょう。

---

## 付録: サンプルアプリのコード詳細

本記事で引用したすべてのコードは `svelte-ai-demo` プロジェクトに含まれています。ここでは、プロジェクトのフォルダ構成、各ファイルの役割、データの流れ、そしてコードリーディングの推奨順序を解説します。

### フォルダ構成（ツリー図）

```
svelte-ai-demo/
├── src/
│   ├── app.html                  … HTML シェル（エントリーポイント）
│   ├── app.d.ts                  … グローバル型定義
│   ├── lib/
│   │   ├── index.ts              … $lib エイリアスのエントリー（空）
│   │   ├── assets/
│   │   │   └── favicon.svg       … ファビコン画像
│   │   ├── components/
│   │   │   ├── Counter.svelte    … カウンターコンポーネント
│   │   │   ├── TodoForm.svelte   … タスク追加フォーム
│   │   │   ├── TodoItem.svelte   … タスク1件の表示・操作
│   │   │   ├── StatsPanel.svelte … 統計パネル＋フィルター
│   │   │   └── ThemeToggle.svelte… ダークモード切替ボタン
│   │   └── stores/
│   │       ├── todo.svelte.ts    … タスクの共有ステート（状態管理）
│   │       └── theme.svelte.ts   … テーマの共有ステート（副作用管理）
│   └── routes/
│       ├── +layout.svelte        … 全ページ共通レイアウト
│       └── +page.svelte          … メインページ（アプリ本体）
├── static/
│   └── robots.txt                … クローラー制御
├── package.json                  … 依存関係・スクリプト定義
├── svelte.config.js              … SvelteKit 設定
├── vite.config.ts                … Vite（ビルドツール）設定
└── tsconfig.json                 … TypeScript 設定
```

### 各ディレクトリの役割

| ディレクトリ | 役割 | SvelteKit における意味 |
|-------------|------|----------------------|
| `src/` | ソースコード全体のルート | SvelteKit が参照するメインディレクトリ |
| `src/lib/` | 再利用可能なモジュール群 | `$lib` エイリアスで `import { ... } from '$lib/...'` のようにインポート可能。コンポーネントやユーティリティを格納する |
| `src/lib/components/` | UI コンポーネント | `.svelte` ファイルを配置。1ファイル = 1コンポーネントの原則 |
| `src/lib/stores/` | 共有ステート（状態管理） | `.svelte.ts` ファイルを配置。Runes（`$state`、`$derived`）をコンポーネント外で使用可能にする Svelte 5 の特殊なファイル形式 |
| `src/routes/` | ページ定義（ファイルベースルーティング） | ディレクトリ構造がそのまま URL パスにマッピングされる。`+page.svelte` がそのパスのページコンポーネント |
| `static/` | 静的アセット | ビルド時にそのままコピーされる。画像やフォントなど |

### 設定ファイルの役割

| ファイル | 何を設定するか | 補足 |
|---------|-------------|------|
| `package.json` | 依存関係とnpmスクリプト | `svelte@^5.49.2` が Svelte 5 本体。`@sveltejs/kit` が SvelteKit フレームワーク |
| `svelte.config.js` | SvelteKit の設定 | `adapter-auto` を使用（デプロイ先を自動検出）。プロダクション環境に合わせて adapter を差し替える |
| `vite.config.ts` | ビルドツール Vite の設定 | `sveltekit()` プラグインを登録するだけのシンプルな構成 |
| `tsconfig.json` | TypeScript 設定 | SvelteKit が生成する `.svelte-kit/tsconfig.json` を extends。`$lib` などのパスエイリアスは SvelteKit が自動設定 |
| `src/app.d.ts` | アプリ全体のグローバル型 | `App.Error`、`App.Locals` 等の型を拡張できる。サンプルではデフォルトのまま |
| `src/app.html` | HTML テンプレート（シェル） | `%sveltekit.head%` と `%sveltekit.body%` がプレースホルダー。SvelteKit がビルド時にここへコンテンツを注入する |

### 各ソースファイルの詳細解説

#### `src/app.html` — HTML シェル

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    %sveltekit.head%
  </head>
  <body data-sveltekit-preload-data="hover">
    <div style="display: contents">%sveltekit.body%</div>
  </body>
</html>
```

すべてのページの大元となる HTML テンプレートです。SvelteKit は `%sveltekit.head%` に `<link>` や `<script>` タグを、`%sveltekit.body%` にレンダリングされたページコンテンツを注入します。`data-sveltekit-preload-data="hover"` は、リンクにホバーした時点でデータのプリロードを開始する SvelteKit の最適化機能です。

#### `src/routes/+layout.svelte` — 共通レイアウト

```svelte
<script lang="ts">
  import favicon from '$lib/assets/favicon.svg';
  let { children } = $props();
</script>

<svelte:head>
  <link rel="icon" href={favicon} />
</svelte:head>

{@render children()}
```

**すべてのページに共通で適用されるレイアウト**です。ここで注目すべき Svelte 5 の機能:

- **`$props()`**: 子コンポーネント（ページ）を `children` として受け取る。React の `children` prop に相当
- **`{@render children()}`**: 受け取った子コンテンツをレンダリング。Svelte 5 の Snippets/render 構文
- **`<svelte:head>`**: `<head>` タグ内に要素を動的に追加する Svelte 固有の特殊要素

#### `src/routes/+page.svelte` — メインページ（アプリのエントリーポイント）

```svelte
<script lang="ts">
  import Counter from '$lib/components/Counter.svelte';
  import TodoForm from '$lib/components/TodoForm.svelte';
  import TodoItem from '$lib/components/TodoItem.svelte';
  import StatsPanel from '$lib/components/StatsPanel.svelte';
  import ThemeToggle from '$lib/components/ThemeToggle.svelte';
  import { getTodos } from '$lib/stores/todo.svelte';

  let mounted = $state(false);
  $effect(() => {
    mounted = true;
    console.log('App mounted - タスク管理アプリが起動しました');
  });
</script>
```

アプリの **ルートページ** です。URL `/` にアクセスしたときにこのファイルがレンダリングされます。

- 5つのコンポーネントをインポートし、セクションごとに配置
- `$state(false)` + `$effect` で マウント検知 → CSS トランジション (`opacity: 0` → `1`) を発動
- `{#each getTodos() as todo (todo.id)}` で todo 一覧をループ描画。`(todo.id)` は React の `key` に相当するキーブロック
- `<style>` ブロックで CSS 変数（`--primary`、`--bg` 等）を `:global(:root)` に定義し、全コンポーネントで参照可能にしている

#### `src/lib/stores/todo.svelte.ts` — タスクの共有ステート

```ts
export interface Todo {
  id: number;
  text: string;
  done: boolean;
  priority: 'high' | 'medium' | 'low';
  createdAt: Date;
}

let todos = $state<Todo[]>([...]);
let nextId = $state(4);
let filter = $state<'all' | 'active' | 'done'>('all');

const filteredTodos = $derived(/* ... */);
const stats = $derived({ total: ..., done: ..., active: ..., highPriority: ... });
```

このファイルは**アプリ全体の状態管理の中核**です。`.svelte.ts` という拡張子が重要で、Svelte 5 のコンパイラがこのファイル内の `$state` / `$derived` を処理します（通常の `.ts` ファイルでは Runes は使えません）。

| エクスポート | 種類 | 説明 |
|------------|------|------|
| `Todo` | interface | タスクの型定義 |
| `getTodos()` | 関数 → `$derived` | フィルター適用済みのタスク一覧を返す |
| `getStats()` | 関数 → `$derived` | 集計値（全件数・完了数・未完了数・高優先度数）を返す |
| `getFilter()` | 関数 → `$state` | 現在のフィルター状態を返す |
| `setFilter(f)` | 関数 | フィルターを変更する |
| `addTodo(text, priority)` | 関数 | タスクを追加する |
| `toggleTodo(id)` | 関数 | タスクの完了/未完了を切り替える |
| `removeTodo(id)` | 関数 | タスクを削除する |

状態（`todos`、`filter`）はモジュールスコープの変数として宣言されており、このファイルをインポートするすべてのコンポーネントが同じ状態を共有します。React で同等のことをするには、Context API + useReducer や外部ライブラリ（Zustand 等）が必要になります。

#### `src/lib/stores/theme.svelte.ts` — テーマの共有ステート

```ts
let dark = $state(false);

$effect.root(() => {
  $effect(() => {
    document.documentElement.classList.toggle('dark', dark);
    localStorage.setItem('theme', dark ? 'dark' : 'light');
  });
});

export function isDark() { return dark; }
export function toggleTheme() { dark = !dark; }
```

ダークモードの状態管理と、DOM / localStorage への副作用をまとめたファイルです。

- **`$effect.root`**: コンポーネント外で `$effect` を使用するために必要なラッパー。コンポーネントの `<script>` 内では不要
- **`$effect`**: `dark` の値が変わるたびに自動実行。`document.documentElement` にクラスを付け外しし、`localStorage` に永続化する

#### `src/lib/components/Counter.svelte` — カウンター

```svelte
<script lang="ts">
  let count = $state(0);
  let doubled = $derived(count * 2);
  let quadrupled = $derived(doubled * 2);
</script>
```

**デモする Svelte 5 機能**: `$state`（リアクティブ変数）、`$derived`（算出値のチェーン）

`count` を変更すると `doubled` → `quadrupled` が自動再計算される。算出値の連鎖（`doubled` に依存する `quadrupled`）も依存配列なしで正しく動作するのがポイントです。

#### `src/lib/components/TodoForm.svelte` — タスク追加フォーム

```svelte
<input type="text" bind:value={newText} placeholder="新しいタスクを入力..." />
<select bind:value={priority}>
  <option value="high">高</option>
  ...
</select>
<button type="submit" disabled={!newText.trim()}>追加</button>
```

**デモする Svelte 5 機能**: `bind:value`（双方向バインディング）、`$state`

- `bind:value={newText}`: ユーザーの入力が即座に `newText` に反映され、`newText` の変更も入力欄に反映される
- `disabled={!newText.trim()}`: `newText` が空のとき自動的にボタンが無効化。`$state` のリアクティビティにより再計算される
- `handleSubmit` 内で `newText = ''` と直接代入するだけでフォームがクリアされる（React のような `setNewText('')` は不要）

#### `src/lib/components/TodoItem.svelte` — タスク1件の表示

```svelte
<script lang="ts">
  import { slide } from 'svelte/transition';
  let { todo }: { todo: Todo } = $props();
</script>

<li class="todo-item" class:done={todo.done} transition:slide={{ duration: 200 }}>
  ...
</li>
```

**デモする Svelte 5 機能**: `$props()`、`transition:`、`class:`

- **`$props()`**: 親コンポーネントから渡される props を受け取る。TypeScript の型注釈でそのまま型安全
- **`transition:slide`**: 要素が DOM に追加/削除されるときにスライドアニメーションを自動適用。`svelte/transition` からインポート
- **`class:done={todo.done}`**: `todo.done` が `true` のとき `done` クラスを付与。CSS で `.todo-item.done { opacity: 0.5; }` のようにスタイルを適用

#### `src/lib/components/StatsPanel.svelte` — 統計パネル

```svelte
{#snippet statCard(label: string, value: number, color: string)}
  <div class="stat-card" style="--accent: {color}">
    <span class="stat-value">{value}</span>
    <span class="stat-label">{label}</span>
  </div>
{/snippet}

{@render statCard('全タスク', getStats().total, '#6366f1')}
```

**デモする Svelte 5 機能**: Snippets（`{#snippet}` / `{@render}`）、`class:` ディレクティブ

- **Snippets**: 型付きの引数を持つ再利用可能なテンプレートブロック。コンポーネントに分離するほどでもない繰り返しパターンに最適
- 4枚の統計カードを同じ Snippet で描画し、フィルターボタンで表示を切り替える

#### `src/lib/components/ThemeToggle.svelte` — テーマ切替ボタン

```svelte
<script lang="ts">
  import { isDark, toggleTheme } from '$lib/stores/theme.svelte';
</script>

<button class="theme-toggle" onclick={toggleTheme}>
  {isDark() ? '☀️' : '🌙'}
</button>
```

**デモする Svelte 5 機能**: 共有ステートの参照（`.svelte.ts` からのインポート）

`theme.svelte.ts` の `isDark()` がリアクティブに追跡されるため、`toggleTheme()` を呼ぶとボタンのアイコンが即座に切り替わります。コンポーネント自体にはステートを持たず、共有ステートに完全に委譲している点が特徴です。

### データフロー図

```
┌─────────────────────────────────────────────────────────────┐
│  共有ステート層（src/lib/stores/）                            │
│                                                             │
│  todo.svelte.ts                    theme.svelte.ts          │
│  ┌─────────────────────┐          ┌──────────────────┐     │
│  │ $state: todos       │          │ $state: dark      │     │
│  │ $state: filter      │          │                   │     │
│  │ $state: nextId      │          │ $effect → DOM     │     │
│  │                     │          │ $effect → storage  │     │
│  │ $derived: filtered  │          └────────┬─────────┘     │
│  │ $derived: stats     │                   │               │
│  └──────────┬──────────┘                   │               │
└─────────────┼──────────────────────────────┼───────────────┘
              │ import                       │ import
              ▼                              ▼
┌─────────────────────────────────────────────────────────────┐
│  ページ層（src/routes/+page.svelte）                         │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  ┌───────────┐ │
│  │ Counter  │  │StatsPanel│  │ TodoForm  │  │ThemeToggle│ │
│  │          │  │          │  │           │  │           │ │
│  │ 独立した  │  │getStats()│  │ addTodo() │  │isDark()   │ │
│  │ $state   │  │getFilter│  │bind:value │  │toggleTheme│ │
│  └──────────┘  │setFilter│  └─────┬─────┘  └───────────┘ │
│                └─────┬────┘        │                       │
│                      │             │                       │
│                      ▼             ▼                       │
│                ┌──────────────────────┐                    │
│                │     TodoItem ×N      │                    │
│                │                      │                    │
│                │  $props: todo        │                    │
│                │  toggleTodo()        │                    │
│                │  removeTodo()        │                    │
│                │  transition:slide    │                    │
│                └──────────────────────┘                    │
└─────────────────────────────────────────────────────────────┘
```

**データの流れのポイント**:
1. `todo.svelte.ts` と `theme.svelte.ts` が **Single Source of Truth**（信頼できる唯一の情報源）
2. 各コンポーネントは store の関数をインポートして **読み取り**（`getTodos()`、`getStats()`）と **書き込み**（`addTodo()`、`toggleTodo()`）を行う
3. `Counter` だけは共有ステートを使わず、コンポーネントローカルの `$state` で完結している（独立したデモ用）
4. `$derived` による算出値は、依存する `$state` が変化したとき自動的に再計算 → 参照しているコンポーネントが自動で再描画される

### コードリーディングの推奨順序

初めてこのプロジェクトを読む場合は、以下の順序をおすすめします。

| 順番 | ファイル | 理由 |
|-----|---------|------|
| 1 | `src/app.html` | アプリの大枠を理解する。SvelteKit の `%sveltekit.body%` がどこに注入されるかを把握 |
| 2 | `src/routes/+layout.svelte` | 全ページ共通の構造を確認。`$props()` と `{@render children()}` の基本形 |
| 3 | `src/routes/+page.svelte` | メインページ。ここでどのコンポーネントが使われているか全体像を把握 |
| 4 | `src/lib/components/Counter.svelte` | 最もシンプルなコンポーネント。`$state` と `$derived` の基本を理解 |
| 5 | `src/lib/stores/todo.svelte.ts` | 共有ステートの実装。`$state`、`$derived` がコンポーネント外でどう機能するかを理解 |
| 6 | `src/lib/components/TodoForm.svelte` | `bind:value` の双方向バインディング。store への書き込み（`addTodo`）の流れ |
| 7 | `src/lib/components/TodoItem.svelte` | `$props()`、`transition:slide`、`class:` の3機能を一度に確認 |
| 8 | `src/lib/components/StatsPanel.svelte` | Snippets（`{#snippet}` / `{@render}`）。store の読み取り（`getStats()`）の流れ |
| 9 | `src/lib/stores/theme.svelte.ts` | `$effect` による副作用管理。コンポーネント外の `$effect.root` の使い方 |
| 10 | `src/lib/components/ThemeToggle.svelte` | 共有ステートのみに依存する最小コンポーネントの形 |

### Svelte 5 の機能と対応ファイルの逆引き表

特定の Svelte 5 機能を学びたい場合は、以下の表から該当ファイルを参照してください。

| Svelte 5 の機能 | 対応ファイル | 補足 |
|-----------------|------------|------|
| `$state` | `Counter.svelte`、`TodoForm.svelte`、`todo.svelte.ts` | コンポーネント内・外の両方で使用 |
| `$derived` | `Counter.svelte`、`todo.svelte.ts` | 算出値チェーン、オブジェクトの算出 |
| `$effect` | `+page.svelte`、`theme.svelte.ts` | コンポーネント内の `$effect` と、コンポーネント外の `$effect.root` |
| `$props()` | `+layout.svelte`、`TodoItem.svelte` | children の受け取り、型付き props |
| `bind:` | `TodoForm.svelte` | `bind:value` による双方向バインディング |
| `transition:` | `TodoItem.svelte` | `transition:slide` によるスライドアニメーション |
| `class:` | `TodoItem.svelte`、`StatsPanel.svelte`、`+page.svelte` | 条件付きクラス適用 |
| Snippets (`{#snippet}` / `{@render}`) | `StatsPanel.svelte`、`+layout.svelte` | 再利用可能なテンプレートブロック |
| `.svelte.ts` ユニバーサルリアクティビティ | `todo.svelte.ts`、`theme.svelte.ts` | コンポーネント外でのリアクティブ状態定義 |
| スコープ付き CSS | 全 `.svelte` ファイル | `<style>` が自動スコープ化 |
| `{#each}` ループ | `+page.svelte` | キー付きリストレンダリング |

---

## サンプルアプリの実行方法

```bash
cd svelte-ai-demo
npm install
npm run dev
```

ブラウザで `http://localhost:5173` にアクセスしてください。

その他の便利なコマンド:

```bash
npm run build        # プロダクションビルド
npm run preview      # ビルド結果をローカルで確認
npm run check        # Svelte / TypeScript の型チェック
```

---

## 参考リンク

- [Svelte 公式サイト](https://svelte.dev/)
- [Svelte 5 リリースブログ "Svelte 5 is alive"](https://svelte.dev/blog/svelte-5-is-alive)
- [Svelte 5 Runes の紹介](https://svelte.dev/blog/runes)
- [Svelte 5 マイグレーションガイド](https://svelte.dev/docs/svelte/v5-migration-guide)
- [Svelte GitHub リリース](https://github.com/sveltejs/svelte/releases)
