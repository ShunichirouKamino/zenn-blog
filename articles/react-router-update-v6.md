---
title: "React Routerのv6へのアップデートが大変だったよという話"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["React", "TypeScript", "ReactRouter"]
published: true
---

# 20 秒で概要

利用中の OSS のアップデートをしている中、React Router の v5 から v6 へのアップデートで少し苦労したというお話です。

## 参考

- [スタッフブログ](https://remix.run/blog/react-router-v6)
- [(公式)UPGRADING TO V6 - Upgrading from v5](https://reactrouter.com/docs/en/v6/upgrading/v5#advantages-of-route-element)

## 修正点を掻い摘んで

- `<Switch>`によるパス分岐は、`<Routes>`になりました。
- `<Route>`のデフォルトが完全一致になったため、exact が不要になりました。
  - `<Switch>` 内の `<Route>` の順序に気をつける必要がなくなりました。
  - `/a`で指定した場合、最初に`/a/b`のようなパスにマッチすることも無くなりました。
- `<Route>`の引数が変わり、`element`にて遷移するコンポーネントを指定するようになりました。
- `<Redirect>`が`<Navigate>`になりました。
  - これにより、デフォルト遷移も replace から push に代わっています。
- `useHistory`が`useNavigate`になりました。
  - これも`<Redirect>`同様にデフォルトが push に代わっています。
- `useRouteMatch`が`useMatch`になりました。
- サブ Route として登録される場合は、絶対パスではなく上位 Route からの相対パスで記載するようになりました。

## 何が困ったの？

後方互換無しの中々に破壊的なアップデートでした。
上記修正点については機械的に置き換えていけばいいものの、機械的にな修正では耐えられなかったポイントについて解説します。

## 注意

- React Router v6 は、React16.8 以降を利用する必要が有ります。

# 困ったポイント

## Routes の直下コンポーネントは Route しか受け付けない

### v5 での実装

これまで、ログイン状況に応じて無理やりログイン画面に戻すために、`<Route>`コンポーネント wrap した`<PrivateRoute>`を作成していました。
中身の処理は外部コンポーネントや自作フックを利用しているので、雰囲気だけでも伝わればよいなと思います。

- ルートコンポーネント

```tsx:index.tsx
<Switch>
  <Route path="/login">
    <Login />
  </Route>
  <PrivateRoute path="/menu">
    <Menu />
  </PrivateRoute>
</Switch>
```

- PrivateRoute コンポーネント

```tsx:index.tsx
export const PrivateRoute: React.FC<RouteProps> = ({
  children,
  ...props
}): JSX.Element => {
  // 自作のuseLoginによるログイン処理の返却値を見て、ログイン状態を確認
  // stateはグローバル状態としてユーザのログイン状態を管理
  const { state } = useLogin();
  // ログイン未済はログイン画面に遷移
  return !state.authenticationStatus ? (
    <Route {...props}>
      <Login />
    </Route>
  ) : (
    // ログイン状態が問題無ければ画面遷移
    <Route {...props}>
      {children}
    </Route>
  );
};
```

### v6 への機械的な修正

まずは機械的に`<Switch>`を`<Route>`に変更し、遷移コンポーネントも`element`で指定するように変更したところ、以下のエラーが発生しました。

- ルートコンポーネント

```tsx:index.tsx
<Routes>
  <Route path="/login" element={<Login />} />
  <PrivateRoute path="/menu" element={<Menu />} />
</Routes>
```

- 発生したエラー

```sh
router.tss:5 Uncaught Error: [PrivateRoute] is not a <Route> component. All component children of <Routes> must be a <Route> or <React.Fragment>
    at invariant (router.tss:5:20)
    at components.tsxx:291:5
    at react.development.jss:1104:1
    at react.development.jss:1067:1
    at mapIntoArray (react.development.jss:964:1)
    at mapIntoArray (react.development.jss:1004:1)
    at mapChildren (react.development.jss:1066:1)
    at Object.forEachChildren [as forEach] (react.development.jss:1103:1)
    at createRoutesFromChildren (components.tsxx:275:3)
    at Routes (components.tsxx:256:20)
```

### v6 への本格対応

最終的に、以下のような冗長な書き方になりました。

- ルートコンポーネント

```tsx:index.tsx
<Routes>
  <Route path="/login" element={<Login />} />
  <Route
    path={screenLocations[MENU_FC]}
    element={
      <PrivateRoute path={screenLocations[MENU_FC]}>
        <Menu />
      </PrivateRoute>
    }
  />
</Routes>
```

## ルートパスへのアクセスで Login 画面にリダイレクトしたい

### v5 での実装

これまでは、以下のように Switch のどこにもマッチしなかった場合には、最終的に`/login` に Redirect するようにしていました。これは、URL 自体も`/`ではなく`/login`としたかったため、Redirect を利用しています。

- ルートコンポーネント

```tsx:index.tsx
<Switch>
  <Route path="/login">
    <Login />
  </Route>
  <Route path="/menu">
    <Menu />
  </Route>
  <Redirect to="/login" />
</Switch>
```

### v6 への機械的な修正

`<Redirect>`は`<Navigate>`となったため以下のように修正すると、前節同様に`<Routes>`配下は`<Route>`にする旨がエラーとして出力されました。（replace, push は意識してません）

```tsx:idnex.tsx
<Routes>
  <Route path="/login">
    <Login />
  </Route>
  <Route path="/menu">
    <Menu />
  </Route>
  <Navigate to="/login" />
</Routes>
```

- エラーログ

```sh
router.tss:5 Uncaught Error: [Navigate] is not a <Route> component. All component children of <Routes> must be a <Route> or <React.Fragment>
    at invariant (router.tss:5:20)
    at components.tsxx:291:5
    at react.development.jss:1104:1
    at react.development.jss:1067:1
    at mapIntoArray (react.development.jss:964:1)
    at mapIntoArray (react.development.jss:1004:1)
    at mapChildren (react.development.jss:1066:1)
    at Object.forEachChildren [as forEach] (react.development.jss:1103:1)
    at createRoutesFromChildren (components.tsxx:275:3)
    at Routes (components.tsxx:256:20)
```

### v6 への本格対応

最終的に、以下のような書き方としました。

```tsx:idnex.tsx
<Routes>
  <Route path="/login" element={<Login />} />
  <Route path="/menu" element={<Menu />} />
  <Route
    path="/"
    element={<Navigate to={screenLocations[LOGIN_FC]} />}
  />
</Routes>
```

## Routes のネストの際にワイルドカードにて子 Route の登録が求められるようになった

### v5 での実装

- ルートコンポーネントでは、Switch にてパスとマッピングして子コンポーネントを登録

```tsx:index.tsx
<Switch>
  <Route path="/login" exact>
    <Login />
  </Route>
  <Route path="/menu">
    <Menu />
  </Route>
  <Route path="/user">
    <Users />
  </Route>
</Switch>
```

- 各子コンポーネント内での詳細なルーティング
  - Users コンポーネントにて UserList と UserDetail を表示するコンポーネント

```tsx:index.tsx
<Switch>
  <Route path="/user" exact>
    <UserList />
  </Route>
  <Route path="/user/details">
    <User />
  </Route>
</Switch>
```

### v6 への機械的な修正

- ルートコンポーネントでは、Switch を Routes に変更し、elements 属性にコンポーネントを登録していきます。

```tsx:index.tsx
<Routes>
  <Route path="/login" element={<Login />} />
  <Route path="/menu" element={<Menu />} />
  <Route path="/user" element={<User />} />>
</Routes>
```

- 子コンポーネントも同様に、Switch を Routes に変更し、elements 属性にコンポーネントを登録していきます。この際、親 Route からの相対パスで設定します。

```tsx:index.tsx
<Routes>
  <Route path="/" elements={<UserList />} />
  <Route path="/details" elements={<User />} />
</Routes>
```

- 発生したエラー

```sh
VM1618 react_devtools_backend.js:3973 You rendered descendant <Routes> (or called `useRoutes()`) at "/workflow" (under <Route path="/user">) but the parent route path has no trailing "*". This means if you navigate deeper, the parent won't match anymore and therefore the child routes will never render.

Please change the parent <Route path="/user"> to <Route path="/user/*">.
```

v6 では、サブルートがレンダリングされる際には、ルートパスにて以降のパスをワイルドカードによって許可する必要が有るようです。
Switch による曖昧一致ではなくなったことや、exact が廃止されたこととにも関連して、正確なルーティングを行うための一環と言えます。

### v6 への本格対応

- 子コンポーネントでサブルートが存在する場合、ルートコンポーネントにワイルドカードを指定します。

```tsx:index.tsx
<Routes>
  <Route path="/login" element={<Login />} />
  <Route path="/menu" element={<Menu />} />
  <Route path="/user/*" element={<User />} />>
</Routes>
```

- 子コンポーネントは変わらず、指定するサブルートはワイルドカード以降のパスを指定しています。

```tsx:index.tsx
<Switch>
  <Route path="/" elements={<UserList />} />
  <Route path="/details" elements={<User />} />
</Switch>
```

# 番外編

update をに合わせてリファクタリングを行う中で、修正が発生したもので、v6 とは関係ないが少しハマったポイントです。

## 現在のパスを取得する方法を useRouteMatch から useLocation に変更したら末尾"/"が消えた

そもそも useLocation から pathname を取得するべきですが、以下のように現在のパスを取得していました。

- 修正前

```tsx:index.tsx
const match = useRouteMatch();
return <Redirect to={`${match.path}details`} />;
```

useRouteMatch が useMatch となったこともあり、現在のパスを取得する目的は改めて useLocation もしくは useHisotry を使うように変更しました。
この際、location.pathname には末尾に`/`が居ないため、以下のように修正しました。

- 修正後

```tsx:index.tsx
const location = useLocation();
return <Navigate to={`${location.pathname}/details`} />
```

※v5 では、useHistory からも history.location.pathname のように現在のパスが取得できていました。useLocation との違いは、useLocation はレンダリング時にミュータブルで有り、useHistory の場合はイミュータブルでした。明確に使い分けなければいけない場面は少なかったですが、useHistory が useNavigate になったことで、実質 useLocation からしか現在のパスは取得できなくなりました。

# 終わりに

なぜこうも破壊的なメジャーアップデートをしたか、これはスタッフブログで述べられています。

> 新しいルーターがリリースされる最大の理由は、React フックの登場です。

クラスベースで React コンポーネントを実装するのはやめて、React フックを利用すた関数コンポーネントでコンポーネントを作成していく潮流に乗っかったということでしょう。

React18 で話題の`<React.Suspense>`や`React.lazy()`との親和性にも触れられており、[サンプルコード](https://github.com/remix-run/react-router/blob/main/examples/lazy-loading/src/App.tsx)も提供されています。

なお、v5 との後方互換も開発中とのことです。
