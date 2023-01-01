---
title: "ReactにReact Flowでネットワークグラフを書いてみた"
emoji: "🐻"
type: "tech"
topics: ["js", "React"]
published: true
---

[https://www.sigmajs.org/](https://reactflow.dev/) でネットワークグラフを描画して見たいと思います。

# 動かしてみたコード

以下のリポジトリのコードで動かしました。docker-composeで動くようになってます。

- https://github.com/k8shiro/sample-reactflow

```
docker-compose run --rm front sh -c "cd app && yarn install"
docker-compose up -d
```

# まとめ

機能全部は試せませんでしたが、一通りネットワークグラフでやりたいようなことができそうに見えました。スマホからもノードを移動したりできたのでtouchイベントも期待通り処理されているのではないかと思います。ただ自動レイアウトはPRO機能みたいだったのではノードの配置は自分でアルゴリズムを実装する必要があるかもしれません。

# install

Reactアプリに

```
yarn add reactflow
```

でvis.jsを追加します。

# 使い方
## 単純なグラフ
ReactFlowコンポーネントにnodes, edgesをわたして描画してます。`proOptions={{ hideAttribution: true }}`はこれを指定しない場合デフォルトではグラフ右下に[React Flow](https://reactflow.dev/)へのリンクが表示されてしまいます。
消す場合は[こちら](https://reactflow.dev/docs/guides/remove-attribution/)を一読ください。


```
import React from 'react';
import ReactFlow from 'reactflow';
import 'reactflow/dist/style.css';

const initialNodes = [
  { id: '1', position: { x: 0, y: 0 }, data: { label: '1' } },
  { id: '2', position: { x: 0, y: 100 }, data: { label: '2' } },
];

const initialEdges = [{ id: 'e1-2', source: '1', target: '2' }];

function SimpleGraph() {
  return (
    <div style={{width: "500px", height: "500px"}}>
      <h1>単純なグラフ</h1>
      <ReactFlow
        nodes={initialNodes}
        edges={initialEdges}
        proOptions={{ hideAttribution: true }}
      >
      </ReactFlow>
    </div>
  );
}

export default SimpleGraph;
```

## ノードを動かせるグラフ
useNodesState, useEdgesStateを使った実装でonNodesChange, onEdgesChangeをReactFlowコンポーネントに渡す事でノードをドラッグできます。

```
import React from 'react';
import ReactFlow, {
  useNodesState,
  useEdgesState,
} from 'reactflow';
import 'reactflow/dist/style.css';

const initialNodes = [
  { id: '1', position: { x: 0, y: 0 }, data: { label: '1' } },
  { id: '2', position: { x: 0, y: 100 }, data: { label: '2' } },
];

const initialEdges = [{ id: 'e1-2', source: '1', target: '2' }];

function MoveableGraph() {
  const [nodes, setNodes, onNodesChange] = useNodesState(initialNodes);
  const [edges, setEdges, onEdgesChange] = useEdgesState(initialEdges);

  return (
    <div style={{width: "500px", height: "500px"}}>
      <h1>動かせるグラフ</h1>
      <ReactFlow
        nodes={nodes}
        edges={edges}
        onNodesChange={onNodesChange}
        onEdgesChange={onEdgesChange}
        proOptions={{ hideAttribution: true }}
      >
      </ReactFlow>
    </div>
  );
}

export default MoveableGraph;
```

## エッジを追加できるグラフ
さらにonConnectを追加する事でノード上の接点をクリックし別のノードの接点へドラッグするとエッジが追加されます。

```
import React from 'react';
import { useCallback } from 'react';
import ReactFlow, {
  Edge,
  Connection,
  useNodesState,
  useEdgesState,
  addEdge,
} from 'reactflow';
import 'reactflow/dist/style.css';

const initialNodes = [
  { id: '1', position: { x: 0, y: 0 }, data: { label: '1' } },
  { id: '2', position: { x: 0, y: 100 }, data: { label: '2' } },
];

const initialEdges = [{ id: 'e1-2', source: '1', target: '2' }];

function EdgeAddableGraph() {
  const [nodes, setNodes, onNodesChange] = useNodesState(initialNodes);
  const [edges, setEdges, onEdgesChange] = useEdgesState(initialEdges);

  const onConnect = useCallback((params : Edge | Connection) => setEdges((eds) => addEdge(params, eds)), [setEdges]);

  return (
    <div style={{width: "500px", height: "500px"}}>
      <h1>エッジを追加できるグラフ</h1>
      <ReactFlow
        nodes={nodes}
        edges={edges}
        onNodesChange={onNodesChange}
        onEdgesChange={onEdgesChange}
        onConnect={onConnect}
        proOptions={{ hideAttribution: true }}
      >
      </ReactFlow>
    </div>
  );
}

export default EdgeAddableGraph;
```

## エッジ・ノードを装飾する
エッジ・ノードを装飾できます。

- オプション
    - エッジ：https://reactflow.dev/docs/api/edges/edge-options/
    - ノード：https://reactflow.dev/docs/api/nodes/node-options/


```
import React from 'react';
import ReactFlow, {
  useNodesState,
  useEdgesState,
} from 'reactflow';
import 'reactflow/dist/style.css';

const initialNodes = [
  { id: '1', position: { x: 0, y: 0 }, data: { label: '1' } },
  { id: '2', position: { x: 0, y: 100 }, data: { label: '2' } },
  { id: '3', position: { x: 200, y: 100 }, data: { label: '3' }, style: { border: '10px solid #ff0000' } },
];

const initialEdges = [
  { id: 'e1-2', source: '1', target: '2'},
  { id: 'e1-3', source: '1', target: '3' , animated: true, style: { stroke: '#ff0000', strokeWidth: 10 }}
];

function StyledGraph() {
  const [nodes, setNodes, onNodesChange] = useNodesState(initialNodes);
  const [edges, setEdges, onEdgesChange] = useEdgesState(initialEdges);

  return (
    <div style={{width: "500px", height: "500px"}}>
      <h1>グラフのノード・エッジを装飾</h1>
      <ReactFlow
        nodes={nodes}
        edges={edges}
        onNodesChange={onNodesChange}
        onEdgesChange={onEdgesChange}
        proOptions={{ hideAttribution: true }}
      >
      </ReactFlow>
    </div>
  );
}

export default StyledGraph;
```

## 自動レイアウト
PRO機能みたいです。

- https://reactflow.dev/docs/examples/layout/auto-layout/




