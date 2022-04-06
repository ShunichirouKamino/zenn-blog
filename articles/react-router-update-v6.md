---
title: "React Routerのv6へのアップデートが大変だったよという話"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["React", "TypeScript", "ReactRouter"]
published: false
---

# 20 秒で概要

せこせこと利用 OSS のアップデートをしている中、React Router の v5 から v6 へのアップデートでとても苦労したというお話です。

## 参考

- [スタッフブログ](https://remix.run/blog/react-router-v6)

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

# 困ったポイント

## Routes の直下コンポーネントは Route しか受け付けない

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

これを、まずは機械的に`<Switch>`を`<Route>`に変更し、遷移コンポーネントも`element`で指定するように変更したところ、以下のエラーが発生しました。

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

# 番外編

update をに合わせてリファクタリングを行う中で、修正が発生したもので、v6 とは関係ないが少しは待ったポイント。

## 現在のパスを取得する方法を useRouteMatch から useLocation に変更したら末尾"/"が消えた

そもそも useLocation か useHistory から pathname を取得するべきですが、以下のように現在のパスを取得していました。

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
