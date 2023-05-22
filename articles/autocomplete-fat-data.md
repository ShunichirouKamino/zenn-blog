---
title: "10万件のSelectBoxが作りたい"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["React", "TypeScript", "MUI"]
published: true
---


# 10秒で概要
- 10万件のデータをサジェストするAutocompleteなSelectBoxを作りたい。
- しかし、1万件を超えたあたりから通常のAutocompleteではレンダリングに時間がかかる。
- 以下の方針が有る。
  - react-windowによるレンダリング以外の範囲の仮想化
  - フロントエンドではデータを保持せず、入力値に応じてSearchのAPIコールを実施する

# Reactのレンダリングによる課題

Reactのレンダリングは、大まかに以下のフローで行われます。
- Triggering a render
  - 新規画面への描画時、またDOM要素の差分を検出したことをTriggerがとして、レンダリングが発生します。
- Committing to the DOM
  - 描画要素に違いがあるDOM要素のみ、DOMノードを変更します。

Autocompleteで表示するデータである`<li>`要素についても当然DOM要素であるため、描画時にデータが全件一括でレンダリングされます。
データ件数が多い場合は、当然ブラウザの挙動は遅くなります。

# 方針
方針として、以下2パターンの解決方法が考えられます。

- react-window
  - Autocompleteに表示予定のデータを仮想化することで、描画したい範囲のみがレンダリングされます。
  - 例）https://mui.com/material-ui/react-autocomplete/#virtualization
- APIコール
  - 入力値によるサジェスト結果を返却するAPIコールし、サーバサイドから都度データの返却を行います。
  - 例）https://mui.com/material-ui/react-autocomplete/#google-maps-place

以下の後者のデメリットを考慮し、今回はreact-windowを使う方針としました。
- サーバサイドAPIの追加開発が不要
  - サンプルではGoogleMapのAPIが「前方一致」で返却されるAPIとなっている。中間一致等も検討する場合、サーバサイド側での改修が必要となる。
- 入力値を受け付けるまではAPIコールをしないため、初期値が0件になってしまい、入力前にスクロールはできない。
  ![](./gif/data-api.gif)

# react-window
[react-window](https://github.com/bvaughn/react-window)は、レンダリング予定のデータを仮想化することで、膨大な量のデータのレンダリングパフォーマンスを向上させるReactライブラリです。レンダリング範囲（プラスマイナス数件）のデータのみがDOMに展開されるため、レンダリングコストが少なくなります。react-windowは、以下の4つのメインコンポーネントから成ります。

- FixedSizeList
  - 各要素の高さが等しいリスト要素
- VariableSizeList
  - 各要素の高さを変更可能なリスト要素
- FixedSizeGrid
  - 各要素のサイズが等しいGrid要素
- VariableSizeGrid
  - 各要素のサイズを変更可能なGrid要素

例として、10000件のリストを描画すると、以下のような動作になります。前後の要素は事前に数件読み込まれているため、スクロールと描画はほぼ同時となります。しかし離れた要素にアクセスする場合は仮想リストから読み込むため、一瞬ホワイトアウトします。
![](./gif/virtuallist.gif)

今回のユースケースでは、Autocompleteに与えるList要素を作成するため、前者2つのコンポーネントを検証します。

# MUIの参考実装
MUIによるreact-windowを利用した実装は[Virtualization](https://mui.com/material-ui/react-autocomplete/#virtualization)を参考にしました。この実装のうち、オーバースペックである以下の実装を削ります。

- AutocompleteによるgroupByオプション
  - レンダリング要素が多い際に、頭文字のアルファベットを抜き出してグルーピングする。
  - このオプションを利用する場合、ref要素によりgroup項目であるかどうかを判断します。group要素かそうでないかで分岐が発生し、keyを付与したり、高さを変更する実装が必要となり、複雑になる。
- userMediaQuery
  - ユースケースによるが、今回はPC限定のWebアプリであるため、レスポンシブなUIを構築するオプションは不要としました。
- VariableSizeList
  - groupの要素について高さを可変としていますが、このオプションが不要な場合は`FixedSizeList`で問題無いはずです。
  - これにより、各要素の高さをキャッシュするなどの処理が不要。
- randam要素を作っている箇所は不要とし、Story側から与えるようにします。

逆に、以下の要素を追加します。
- 引数をOption型とし、外部から与えられるようにします。
- stateを管理できるようにし、選択されている値を`useState`にて管理します。
- label, valueからなるObjectでの管理をするために、各オプションの値を修正しています。
  - getOptionLabelでは、valueを描画するように修正しました。
  - renderOptionでも、valueを描画するように修正しました。

以下に、シンプルな実装に修正したコードを添付します。

```tsx
import * as React from "react";
import TextField from "@mui/material/TextField";
import Autocomplete, { autocompleteClasses } from "@mui/material/Autocomplete";
import { FixedSizeList, ListChildComponentProps } from "react-window";
import Typography from "@mui/material/Typography";
import Popper from "@mui/material/Popper";
import { styled } from "@mui/material/styles";

const RenderRow: React.FC<ListChildComponentProps> = (
  props: ListChildComponentProps
): JSX.Element => {
  const { data, index, style } = props;
  const dataSet = data[index];

  return (
    // dataSet[0]にはFixedSizeListに引き渡したchildrenの情報が含まれており、
    // Typographyに渡すことでonClickやonMouseOverの情報を引き継ぐ
    <Typography component="li" {...dataSet[0]} noWrap style={style}>
      {`#${dataSet[2] + 1} - ${dataSet[1]}`}
    </Typography>
  );
};

const OuterElementContext = React.createContext({});

const OuterElementType = React.forwardRef<HTMLDivElement>((props, ref) => {
  const outerProps = React.useContext(OuterElementContext);
  return <div ref={ref} {...props} {...outerProps} />;
});

/**
 * react-windowとのadapterコンポーネント
 */
const ListboxComponent = React.forwardRef<
  HTMLDivElement,
  React.HTMLAttributes<HTMLElement>
>((props, ref) => {
  const { children, ...other } = props;
  const itemData: React.ReactElement[] = [];
  (children as React.ReactElement[]).forEach(
    // itemはReact要素 or childrenが存在する場合はReact.ReactElement[]要素である
    (item: React.ReactElement & { children?: React.ReactElement[] }) => {
      itemData.push(item);
      itemData.push(...(item.children || []));
    }
  );

  const itemCount = itemData.length;
  const itemSize = 48;

  /**
   * サジェスト結果に応じてサイズを可変とする関数.
   * 8件以下になった場合は、要素数に応じて収縮.
   */
  const getHeight = () => {
    if (itemCount > 8) {
      return 8 * itemSize;
    }
    return itemCount * itemSize;
  };

  return (
    <div ref={ref} style={{ backgroundColor: "#eee" }}>
      <OuterElementContext.Provider value={other}>
        <FixedSizeList
          itemData={itemData}
          height={getHeight() + 2}
          width="100%"
          outerElementType={OuterElementType}
          innerElementType="ul"
          itemSize={30}
          overscanCount={5}
          itemCount={itemCount}
        >
          {RenderRow}
        </FixedSizeList>
      </OuterElementContext.Provider>
    </div>
  );
});

export type Option<V> = {
  label: string;
  value: V;
};

type DataListVirtualizationProps<V> = {
  options: Option<V>[];
};

const StyledPopper = styled(Popper)({
  [`& .${autocompleteClasses.listbox}`]: {
    boxSizing: "border-box",
    "& ul": {
      padding: 0,
      margin: 0,
    },
  },
});

export function DataListVirtualization<V>({
  options,
}: DataListVirtualizationProps<V>): JSX.Element {
  const [option, setOption] = React.useState<Option<V> | null>(null);
  console.log(option);
  return (
    <Autocomplete
      sx={{ width: 300 }}
      value={option}
      onChange={(
        _event: React.SyntheticEvent<Element, Event>,
        newValue: Option<V> | null
      ) => {
        setOption(newValue);
      }}
      disableListWrap
      ListboxComponent={ListboxComponent}
      PopperComponent={StyledPopper}
      options={options}
      renderInput={(params) => (
        <TextField {...params} label="100,000 options" />
      )}
      getOptionLabel={(option) => {
        return option.value as string;
      }}
      renderOption={(props, option, state) =>
        [props, `${option.value}`, state.index] as React.ReactNode
      }
    />
  );
}

export default DataListVirtualization;
```

Storyから要素数を変更して与えた場合の動きは、以下の通りです。

![](./gif/virtualdatalist-paform.gif)

# TIPS
- レンダリングの前後読み込まれることで、マウススクロール時には滑らかなレンダリングが行われるようになっています。前後訳5件のデータを読み込んでおり、デフォルト値を変えられるのかどうかは未検証です。
  ![](./gif/indexlist.gif)
- 20万件を超えたあたりから、レンダリング遅延が発生します。これは、Autocomplete側のlabelを構築する際に、1件ごとに処理が走るためです。
  - `getOptionLabel`オプションにより、labelを構築するコールバック関数を指定することができます。これはデフォルトでも`(option) => option.label ?? option`となっております。この処理は、Optionの描画範囲とは関係なく、Optionの描画されるタイミングで、仮想化されたデータも含めて全件のlabelを構築するような動きをします。
  - これにより、これ以上の件数をレンダリングする際や、画面上複数のAutocompleteが必要になる際は、APIコールによる手法も検討する必要が有ります。

# まとめ
- 膨大な件数を扱うAutocompleteについて、以下の2種類の方法があります。
  - APIコールにより、入力値からサジェスト結果を返却する方法。
  - react-windowにより、レンダリング以外の範囲を仮想化しておく方法。
- react-windowによる実装では、10万件程度の描画であれば滑らかなレンダリングが行えます。しかしそれ以上の件数となる場合はAutocomplete側が仮想化のことを考慮しているわけでは無いので、APIコールをする方法を検討する必要が有ります。
  - `getOptionLabel`の中に`console.log`でも仕込もうものなら、膨大な処理時間がかかります。