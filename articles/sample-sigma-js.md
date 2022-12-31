---
title: "Reactにsigmaでネットワークグラフを書いてみた"
emoji: "🐻"
type: "tech"
topics: ["js", "React"]
published: false # 公開設定（falseにすると下書き）
---

https://www.sigmajs.org/ でネットワークグラフを描画して見たいと思います。
今回はReact上で使用したいのでreact-sigmaの方を使って行きます。

公式：https://sim51.github.io/react-sigma/

# 動かしてみたコード

以下のリポジトリのコードで動かしました。docker-composeで動くようになってます。

- https://github.com/k8shiro/sample-sigma.js

```
docker-compose run --rm front sh -c "cd app && yarn install"
docker-compose up -d
```

# まとめ

- Reactから使える
- 座標指定配置・自動配置ができる
- ノード・エッジにラベルが貼れる
- 色や大きさを変更できる
- ノードをクリックしてイベントを起こせる

という点では問題なく使えそうでした。

- ノードをマウスで移動できる

という点のみ、mouseイベントは期待した挙動をしますが、touchイベントがうまく動かせませんでした。どうにか実装すれば解消しそうではあるのですが、公式のサンプル自体がスマホから上手く扱えなかったので、PCからは充分だが少し残念というのが検証時点での感想になります。

# Install
Reactアプリに

```
yarn add @react-sigma/core sigma graphology graphology-types lodash
```

でsigmajs.jsを追加します。

# 使い方
## 単純なグラフ
基本的にはSigmaContainerコンポーネントにMultiDirectedGraphで作ったgraphを渡すことで描画できるようです。

```
import React from 'react';
import "@react-sigma/core/lib/react-sigma.min.css";
import { MultiDirectedGraph } from "graphology";
import { SigmaContainer } from "@react-sigma/core";

function SimpleGraphA() {
  // グラフ生成
  const graph = new MultiDirectedGraph();
  // ノード追加
  graph.addNode("A", { x: 0, y: 0, label: "Node A", size: 10 });
  graph.addNode("B", { x: 1, y: 1, label: "Node B", size: 10 });
  // エッジ追加
  graph.addEdgeWithKey("rel1", "A", "B", { label: "REL_1" });

  return (
    <div className="SimpleGraphA">
      <h1>単純なグラフ</h1>
      <SigmaContainer style={{ height: "500px" }} graph={graph}></SigmaContainer>
    </div>
  );
}

export default SimpleGraphA;
```

## useLoadGraphを使って描画する

```
import React, { FC, useEffect } from "react";

import "@react-sigma/core/lib/react-sigma.min.css";
import { MultiDirectedGraph } from "graphology";
import { SigmaContainer, useLoadGraph } from "@react-sigma/core";

const LoadGraphWithHook: FC = () => {
  const MyGraph: FC = () => {
    const loadGraph = useLoadGraph();

    useEffect(() => {
      // Create the graph
      const graph = new MultiDirectedGraph();
      graph.addNode("A", { x: 0, y: 0, label: "Node A", size: 10 });
      graph.addNode("B", { x: 1, y: 1, label: "Node B", size: 10 });
      graph.addEdgeWithKey("rel1", "A", "B", { label: "REL_1" });
      loadGraph(graph);
    }, [loadGraph]);

    return null;
  };

  return (
    <div className="LoadGraphWithHook">
      <h1>useLoadGraphを使ったグラフ</h1>
      <SigmaContainer style={{ height: "500px" }}>
        <MyGraph />
      </SigmaContainer>
    </div>
  );
};

export default LoadGraphWithHook;
```

## グラフの設定を変える
SigmaContainerにgraphとsettingsを渡すことで、グラフの種類や設定を変更できます。
以下はMultiDirectedGraphを渡す事でノード間のエッジの重複を許可し、エッジの形とラベル表示を変更しています。


```
import React, { FC, useEffect } from "react";

import "@react-sigma/core/lib/react-sigma.min.css";
import { MultiDirectedGraph } from "graphology";
import { SigmaContainer, useLoadGraph } from "@react-sigma/core";

export const MultiGraph: FC = () => {
  const MyGraph: FC = () => {
    const loadGraph = useLoadGraph();

    useEffect(() => {
      // Create the graph
      const graph = new MultiDirectedGraph();
      graph.addNode("A", { x: 0, y: 0, label: "Node A", size: 10 });
      graph.addNode("B", { x: 1, y: 1, label: "Node B", size: 10 });
      graph.addEdgeWithKey("rel1", "A", "B", { label: "REL_1" });
      graph.addEdgeWithKey("rel2", "A", "B", { label: "REL_2" });
      loadGraph(graph);
    }, [loadGraph]);

    return null;
  };

  return (
    <SigmaContainer
      graph={MultiDirectedGraph}
      style={{ height: "500px" }}
      settings={{ renderEdgeLabels: true, defaultEdgeType: "arrow" }}
    >
      <MyGraph />
    </SigmaContainer>
  );
};

export default MultiGraph;
```



## グラフ上のイベントを拾う
先ほどのSigmaContainerコンポーネントの子供にGraphEventsコンポーネントが追加されています。グラフ上でのマウス操作がconsole.logに出力されます。


```
import React, { FC, useEffect } from "react";

import "@react-sigma/core/lib/react-sigma.min.css";
import { SigmaContainer, useRegisterEvents, useLoadGraph } from "@react-sigma/core";
import { MultiDirectedGraph } from "graphology";

//import { GraphDefault } from "../GraphDefault";

const GraphEventsLog: FC = () => {
  const MyGraph: FC = () => {
    const loadGraph = useLoadGraph();

    useEffect(() => {
        // Create the graph
        var nodes = ['A', 'B', 'C', 'D'];

        const graph = new MultiDirectedGraph();
        nodes.forEach((n, idx) => {
          const x = Math.floor(idx / 2);
          const y = idx % 2;
          graph.addNode(n, { x: x, y: y, label: "Node " + n, size: 10 });
        })

        for (const l of nodes) {
          for (const r of nodes) {
            graph.addEdgeWithKey("rel" + l + "_" + r, l, r, { label: "REL" + l + "_" + r});
          }
        }
        loadGraph(graph);
    }, [loadGraph]);

    return null;
    };
  const GraphEvents: React.FC = () => {
    const registerEvents = useRegisterEvents();

    useEffect(() => {
      // Register the events
      registerEvents({
        // node events
        clickNode: (event) => console.log("clickNode", event.event, event.node, event.preventSigmaDefault),
        doubleClickNode: (event) => console.log("doubleClickNode", event.event, event.node, event.preventSigmaDefault),
        rightClickNode: (event) => console.log("rightClickNode", event.event, event.node, event.preventSigmaDefault),
        wheelNode: (event) => console.log("wheelNode", event.event, event.node, event.preventSigmaDefault),
        downNode: (event) => console.log("downNode", event.event, event.node, event.preventSigmaDefault),
        enterNode: (event) => console.log("enterNode", event.node),
        leaveNode: (event) => console.log("leaveNode", event.node),
        // edge events
        clickEdge: (event) => console.log("clickEdge", event.event, event.edge, event.preventSigmaDefault),
        doubleClickEdge: (event) => console.log("doubleClickEdge", event.event, event.edge, event.preventSigmaDefault),
        rightClickEdge: (event) => console.log("rightClickEdge", event.event, event.edge, event.preventSigmaDefault),
        wheelEdge: (event) => console.log("wheelEdge", event.event, event.edge, event.preventSigmaDefault),
        downEdge: (event) => console.log("downEdge", event.event, event.edge, event.preventSigmaDefault),
        enterEdge: (event) => console.log("enterEdge", event.edge),
        leaveEdge: (event) => console.log("leaveEdge", event.edge),
        // stage events
        clickStage: (event) => console.log("clickStage", event.event, event.preventSigmaDefault),
        doubleClickStage: (event) => console.log("doubleClickStage", event.event, event.preventSigmaDefault),
        rightClickStage: (event) => console.log("rightClickStage", event.event, event.preventSigmaDefault),
        wheelStage: (event) => console.log("wheelStage", event.event, event.preventSigmaDefault),
        downStage: (event) => console.log("downStage", event.event, event.preventSigmaDefault),
        // default mouse events
        click: (event) => console.log("click", event.x, event.y),
        doubleClick: (event) => console.log("doubleClick", event.x, event.y),
        wheel: (event) => console.log("wheel", event.x, event.y, event.delta),
        rightClick: (event) => console.log("rightClick", event.x, event.y),
        mouseup: (event) => console.log("mouseup", event.x, event.y),
        mousedown: (event) => console.log("mousedown", event.x, event.y),
        mousemove: (event) => console.log("mousemove", event.x, event.y),
        // default touch events
        touchup: (event) => console.log("touchup", event.touches),
        touchdown: (event) => console.log("touchdown", event.touches),
        touchmove: (event) => console.log("touchmove", event.touches),
        // sigma kill
        kill: () => console.log("kill"),
        // sigma camera update
        updated: (event) => console.log("updated", event.x, event.y, event.angle, event.ratio),
      });
    }, [registerEvents]);

    return null;
  };
  return (
    <div className="GraphEventsLog">
      <h1>グラフのイベント(console.logに出力)</h1>
      <SigmaContainer
        graph={MultiDirectedGraph}
        style={{ height: "500px" }}
        settings={{ renderEdgeLabels: true, defaultEdgeType: "arrow" }}
      >
        <MyGraph />
        <GraphEvents />
      </SigmaContainer>
    </div>
  );
};

export default GraphEventsLog;
```
## ノードをドラッグできるようにする
硬式のsampleコードではtouchイベントも対応していたのですが、どうも上手く動かなかった(公式のデモをスマホから触っても期待したい挙動をしない)のでtouchイベントのコードでは削除してます。

コードとしては、DragEventsコンポーネントにノードをドラッグする機能を追加して、先ほどのSigmaContainerコンポーネントの子供にさらにDragEventsコンポーネントが追加されています。

```
import React, { FC, useEffect, useState } from "react";
import "@react-sigma/core/lib/react-sigma.min.css";
import { SigmaContainer, useRegisterEvents, useLoadGraph, useSigma } from "@react-sigma/core";
import { MultiDirectedGraph } from "graphology";

//import { GraphDefault } from "../GraphDefault";

const DragNdrop: FC = () => {
  const MyGraph: FC = () => {
    const loadGraph = useLoadGraph();

    useEffect(() => {
        // Create the graph
        var nodes = ['A', 'B', 'C', 'D'];

        const graph = new MultiDirectedGraph();
        nodes.forEach((n, idx) => {
          const x = Math.floor(idx / 2);
          const y = idx % 2;
          graph.addNode(n, { x: x, y: y, label: "Node " + n, size: 10 });
        })

        for (const l of nodes) {
          for (const r of nodes) {
            graph.addEdgeWithKey("rel" + l + "_" + r, l, r, { label: "REL" + l + "_" + r});
          }
        }
        loadGraph(graph);
    }, [loadGraph]);

    return null;
    };
  const GraphEvents: React.FC = () => {
    const registerEvents = useRegisterEvents();

    useEffect(() => {
      // Register the events
      registerEvents({
        // node events
        clickNode: (event) => console.log("clickNode", event.event, event.node, event.preventSigmaDefault),
        doubleClickNode: (event) => console.log("doubleClickNode", event.event, event.node, event.preventSigmaDefault),
        rightClickNode: (event) => console.log("rightClickNode", event.event, event.node, event.preventSigmaDefault),
        wheelNode: (event) => console.log("wheelNode", event.event, event.node, event.preventSigmaDefault),
        downNode: (event) => console.log("downNode", event.event, event.node, event.preventSigmaDefault),
        enterNode: (event) => console.log("enterNode", event.node),
        leaveNode: (event) => console.log("leaveNode", event.node),
        // edge events
        clickEdge: (event) => console.log("clickEdge", event.event, event.edge, event.preventSigmaDefault),
        doubleClickEdge: (event) => console.log("doubleClickEdge", event.event, event.edge, event.preventSigmaDefault),
        rightClickEdge: (event) => console.log("rightClickEdge", event.event, event.edge, event.preventSigmaDefault),
        wheelEdge: (event) => console.log("wheelEdge", event.event, event.edge, event.preventSigmaDefault),
        downEdge: (event) => console.log("downEdge", event.event, event.edge, event.preventSigmaDefault),
        enterEdge: (event) => console.log("enterEdge", event.edge),
        leaveEdge: (event) => console.log("leaveEdge", event.edge),
        // stage events
        clickStage: (event) => console.log("clickStage", event.event, event.preventSigmaDefault),
        doubleClickStage: (event) => console.log("doubleClickStage", event.event, event.preventSigmaDefault),
        rightClickStage: (event) => console.log("rightClickStage", event.event, event.preventSigmaDefault),
        wheelStage: (event) => console.log("wheelStage", event.event, event.preventSigmaDefault),
        downStage: (event) => console.log("downStage", event.event, event.preventSigmaDefault),
        // default mouse events
        click: (event) => console.log("click", event.x, event.y),
        doubleClick: (event) => console.log("doubleClick", event.x, event.y),
        wheel: (event) => console.log("wheel", event.x, event.y, event.delta),
        rightClick: (event) => console.log("rightClick", event.x, event.y),
        mouseup: (event) => console.log("mouseup", event.x, event.y),
        mousedown: (event) => console.log("mousedown", event.x, event.y),
        mousemove: (event) => console.log("mousemove", event.x, event.y),
        // default touch events
        touchup: (event) => console.log("touchup", event.touches),
        touchdown: (event) => console.log("touchdown", event.touches),
        touchmove: (event) => console.log("touchmove", event.touches),
        // sigma kill
        kill: () => console.log("kill"),
        // sigma camera update
        updated: (event) => console.log("updated", event.x, event.y, event.angle, event.ratio),
      });
    }, [registerEvents]);

    return null;
  };
  const DragEvents: React.FC = () => {
    const registerEvents = useRegisterEvents();
    const sigma = useSigma();
    const [draggedNode, setDraggedNode] = useState<string | null>(null);

    useEffect(() => {
      // Register the events
      registerEvents({
        downNode: (e) => {
          setDraggedNode(e.node);
          sigma.getGraph().setNodeAttribute(e.node, "highlighted", true);
        },
        mouseup: (e) => {
          if (draggedNode) {
            setDraggedNode(null);
            sigma.getGraph().removeNodeAttribute(draggedNode, "highlighted");
          }
        },
        mousedown: (e) => {
          // Disable the autoscale at the first down interaction
          if (!sigma.getCustomBBox()) sigma.setCustomBBox(sigma.getBBox());
        },
        mousemove: (e) => {
          if (draggedNode) {
            // Get new position of node
            const pos = sigma.viewportToGraph(e);
            sigma.getGraph().setNodeAttribute(draggedNode, "x", pos.x);
            sigma.getGraph().setNodeAttribute(draggedNode, "y", pos.y);

            // Prevent sigma to move camera:
            e.preventSigmaDefault();
            e.original.preventDefault();
            e.original.stopPropagation();
          }
        },
      });
    }, [registerEvents, sigma, draggedNode]);

    return null;
  };
  return (
    <div className="GraphEvents">
      <h1>ノードをドラッグできるグラフ</h1>
      <SigmaContainer
        graph={MultiDirectedGraph}
        style={{ height: "500px" }}
        settings={{ renderEdgeLabels: true, defaultEdgeType: "arrow" }}
      >
        <MyGraph />
        <GraphEvents />
        <DragEvents />
      </SigmaContainer>
    </div>
  );
};

export default DragNdrop;
```
