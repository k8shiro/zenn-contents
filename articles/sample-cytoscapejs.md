---
title: "ReactにCytoscape.jsでネットワークグラフを書いてみた"
emoji: "🐻"
type: "tech"
topics: ["js", "React"]
published: true
---

ReactにCytoscape.jsでネットワークグラフを書いてみた

[https://js.cytoscape.org/](https://js.cytoscape.org/)でネットワークグラフを描画して見たいと思います。

# 動かしてみたコード

以下のリポジトリのコードで動かしました。docker-composeで動くようになってます。

- https://github.com/k8shiro/sample-cytoscape.js

```
docker-compose run --rm front sh -c "cd app && yarn install"
docker-compose up -d
```

# まとめ

一通りネットワークグラフでやりたいようなことができそうに見えました。スマホからもノードを移動したりできたのでtouchイベントも期待通り処理されているのではないかと思います。いくつか他のネットワークグラフ系のパッケージを試して見ましたが、基本的にCytoscape.jsを使っておけば困らないのではないでしょうか。

# install

Reactアプリに

```
yarn add react-cytoscapejs
yarn add cytoscape@3.23.0
yarn add @types/react-cytoscapejs # typescriptのみ
```

でcytoscape.jsを追加します。
cytoscapeは3.2.19以上の必要があり、ここでは記事執筆タイミングで最新の3.23.0を使用しました。

# 使い方

## 単純なグラフ
ノードとエッジを定義したelementsをCytoscapeComponentに渡すだけでグラフが描画されます。特に設定してませんがこの時点でノードをドラッグできます

```
import React from 'react';
import CytoscapeComponent from 'react-cytoscapejs';

function SimpleGraph() {
  const elements = [
    { data: { id: 'one', label: 'Node 1' }, position: { x: 100, y: 100 } },
    { data: { id: 'two', label: 'Node 2' }, position: { x: 200, y: 200 } },
    { data: { source: 'one', target: 'two', label: 'Edge from Node1 to Node2' } }
  ];
  return (
    <div style={{width: "500px", height: "500px"}}>
      <h1>単純なグラフ</h1>
      <CytoscapeComponent elements={elements} style={ { width: '600px', height: '600px' } } />
    </div>
  );
}

export default SimpleGraph;
```

また、同様のグラフを以下のようにCytoscapeComponent.normalizeElementsを使用して描画することもできます。

```
import React from 'react';
import CytoscapeComponent from 'react-cytoscapejs';

function SimpleGraph2() {
  return (
    <div>
      <h1>単純なグラフ2</h1>
      <CytoscapeComponent
        style={{ width: '300px', height: '300px' }}
        elements={CytoscapeComponent.normalizeElements({
          nodes: [
            { data: { id: 'one', label: 'Node 1' }, position: { x: 100, y: 100 } },
            { data: { id: 'two', label: 'Node 2' }, position: { x: 200, y: 200 } }
          ],
          edges: [
            {
              data: { source: 'one', target: 'two', label: 'Edge from Node1 to Node2' }
            }
          ]
        })}
      />
    </div>
  );
}

export default SimpleGraph2;
```

## ノード・エッジのスタイル
以下のようにノード・エッジのスタイルを設定できます。
ただし、この各ノード・エッジごとに設定する方法は他の方法が選択できるなら使用しないほうが良いみたいです。

```
import React from 'react';
import CytoscapeComponent from 'react-cytoscapejs';

function StyledGraph() {
  const elements = [
    { data: { id: 'one', label: 'Node 1' }, position: { x: 100, y: 100 } },
    { data: { id: 'two', label: 'Node 2' }, position: { x: 200, y: 200 } },
    { 
      data: { id: 'three', label: 'Node 3' },
      position: { x: 200, y: 100 },
      style: {
        width: 20,
        height: 20,
        shape: 'rectangle',
        backgroundColor: 'red',
        borderColor: 'black',
        borderWidth: 3
      }
    },
    { data: { source: 'one', target: 'two', label: 'Edge from Node1 to Node2' } },
    { 
      data: { source: 'two', target: 'three', label: 'Edge from Node2 to Node3' },
      style: {
        width: 10,
        lineColor: 'green',
        lineStyle: 'dashed'
      }
    }
  ];
  return (
    <div>
      <h1>スタイルをつけたグラフ</h1>
      <CytoscapeComponent elements={elements} style={{ width: '300px', height: '300px' }} />
    </div>
  );
}

export default StyledGraph;
```

## スタイルをまとめて適用する
以下のように`cy={(cy) => cy.style(cyStylesheet)}`を適用することでまとめて全体にスタイルを適用することもできます。

```
import React from 'react';
import CytoscapeComponent from 'react-cytoscapejs';

function StyledGraph2() {
  const elements = [
    { data: { id: 'one', label: 'Node 1' }, position: { x: 100, y: 100 } },
    { data: { id: 'two', label: 'Node 2' }, position: { x: 200, y: 200 } },
    { data: { id: 'three', label: 'Node 3' }, position: { x: 200, y: 100 }},
    { data: { source: 'one', target: 'two', label: 'Edge from Node1 to Node2' } },
    { data: { source: 'two', target: 'three', label: 'Edge from Node2 to Node3' }}
  ];
  const cyStylesheet=[
    {
      selector: 'node',
      style: {
        label: 'data(label)',
        width: 20,
        height: 20,
        shape: 'rectangle',
        backgroundColor: 'red',
        borderColor: 'black',
        borderWidth: 3
      }
    },
    {
      selector: 'edge',
      style: {
        width: 10,
        lineColor: 'green',
        lineStyle: 'dashed'
      }
    }
  ]
  return (
    <div>
      <h1>全体にスタイルをつけたグラフ</h1>
      <CytoscapeComponent
        elements={elements}
        style={{ width: '300px', height: '300px' }}
        cy={(cy) => cy.style(cyStylesheet)}
      />
    </div>
  );
}

export default StyledGraph2;
```

## selectorでノード・エッジのスタイルを設定する
全体に`cy={(cy) => cy.style(cyStylesheet)}`を適用する時に`cyStylesheet`の`selector`の指定の仕方で個別のノード・エッジを指定してスタイルを適用することもできます。`selector: '#four',`なら`id`が`four`の要素だけ、`selector: '[label="Edge from Node2 to Node4"]'`なら`label`が`Edge from Node2 to Node4`の要素に適用するような書き方になります。



```
import React from 'react';
import CytoscapeComponent from 'react-cytoscapejs';

function StyledGraph3() {
  console.log('StyledGraph3');
  const elements = [
    { data: { id: 'one', label: 'Node 1' }, position: { x: 100, y: 100 } },
    { data: { id: 'two', label: 'Node 2' }, position: { x: 200, y: 200 } },
    { data: { id: 'three', label: 'Node 3' }, position: { x: 200, y: 100 }},
    { data: { id: 'four', label: 'Node 4' }, position: { x: 100, y: 200 }},
    { data: { source: 'one', target: 'two', label: 'Edge from Node1 to Node2' } },
    { data: { source: 'two', target: 'three', label: 'Edge from Node2 to Node3' }},
    { data: { source: 'two', target: 'four', label: 'Edge from Node2 to Node4' }}
  ];
  const cyStylesheet=[
    {
      selector: 'node',
      style: {
        label: 'data(label)',
        width: 20,
        height: 20,
        shape: 'rectangle',
        backgroundColor: 'red',
        borderColor: 'black',
        borderWidth: 3
      }
    },
    {
      selector: 'edge',
      style: {
        width: 10,
        lineColor: 'green',
        lineStyle: 'dashed'
      }
    },
    {
      selector: '#four',
      style: {
        label: 'data(label)',
        width: 20,
        height: 20,
        shape: 'rectangle',
        backgroundColor: 'red',
        borderColor: 'blue',
        borderWidth: 3
      }
    },
    {
      selector: '[label="Edge from Node2 to Node4"]',
      style: {
        width: 10,
        lineColor: 'blue',
        lineStyle: 'dotted'
      }
    },
  ];
  return (
    <div>
      <h1>selectorでスタイルを変える</h1>
      <CytoscapeComponent
        elements={elements}
        style={{ width: '300px', height: '300px' }}
        cy={(cy) => cy.style(cyStylesheet)}
      />
    </div>
  );
}

export default StyledGraph3;
```


## 自動でレイアウトする
layoutを指定して自動でノードを配置できます。

```
import React from 'react';
import CytoscapeComponent from 'react-cytoscapejs';
import { ElementDefinition } from "cytoscape";

function SimpleGraph2() {
  const nodesLength = 10;
  let elements: ElementDefinition[] = [];
  for(let i=0; i < nodesLength; i++){
    elements.push(
      { data: { id: String(i), label: 'Node ' + i }}
    )
  }

  for(let l=0; l < nodesLength; l++){
    for(let r=0; r < nodesLength; r++){
      elements.push(
        { data: { source: l, target: r, label: 'Edge from ' + l + ' to ' + r } }
      )
    }
  }

  const layout1 = { name: 'random' };
  const layout2 = { name: 'grid' };
  const layout3 = { name: 'circle' };
  const layout4 = { name: 'concentric' };
  const layout5 = { name: 'breadthfirst' };
  const layout6 = { name: 'cose' };

  return (
    <div>
      <h1>自動レイアウトしたグラフ</h1>
      random
      <CytoscapeComponent elements={elements} layout={layout1} style={{ width: '300px', height: '300px' }} />
      grid
      <CytoscapeComponent elements={elements} layout={layout2} style={{ width: '300px', height: '300px' }} />
      circle
      <CytoscapeComponent elements={elements} layout={layout3} style={{ width: '300px', height: '300px' }} />
      concentric
      <CytoscapeComponent elements={elements} layout={layout4} style={{ width: '300px', height: '300px' }} />
      breadthfirst
      <CytoscapeComponent elements={elements} layout={layout5} style={{ width: '300px', height: '300px' }} />
      cose
      <CytoscapeComponent elements={elements} layout={layout6} style={{ width: '300px', height: '300px' }} />
    </div>
  );
```
