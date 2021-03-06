graphx-labelpropagation

The basic idea of label propagation is that we start with seed users in a network who have some information about the label probabilities. Following the weights in the network, we propagate the label
probabilities from the seed user into the network users. It will assign label probabilities for those unknown users. It has a wide application, for example, in churn analysis, we want to know how
likely an non-churner will churn if he has a few churner friends.

## Algorithms
The label propagation algorithm is described as follows:
1. Propagate Y = TY
2. Row normalize Y
3. Clamp the labeled data. Repeat from step 1 until Y converges.

#### Usage

Here is a usage example for label propagation:

```Scala
import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD
import org.apache.spark.SparkContext
import org.apache.spark.graphx.Edge
import org.apache.spark.graphx.Graph

val sparkConf = new SparkConf()
val sc = new SparkContext(sparkConf)


val edges = sc.parallelize(Array(
  Edge(1L, 2L, 1.0), Edge(2L, 1L, 3.0),
  Edge(3L, 4L, 1.0), Edge(4L, 5L, 1.0),
  Edge(1L, 5L, 2.0), Edge(5L, 2L, 1.0)
  ))

val users:RDD[(Long, String, Double)] = sc.parallelize(Array(
  (1L, "1", 0.5), (1L, "2", 0.3), (1L, "3", 0.2)
))

val labelsArr = users.map(x => (x._2, 1)).reduceByKey(_+_).map(x => x._1).collect()
val verts = users.map(x => (x._1, (x._2, x._3))).groupByKey().map(x => {
val vert = new LPVertex
   vert.isSeedNode =true
   vert.injectedLabels = x._2.toMap
   (x._1, vert)
 })

val graph = Graph.fromEdges(edges, 1).outerJoinVertices(verts){(vid, vdata, vert) => if(vert eq None){val v = new LPVertex; v} else vert.get}
val lp = new LPZhuGhahramani(graph, undirected=true, labels = labelsArr)
val rankGraph = lp.runUntilConvergence(0.0001)
rankGraph.vertices.collect().foreach(x => 
  println(x._1 + ": " + x._2.estimatedLabels.mkString(","))
)


```