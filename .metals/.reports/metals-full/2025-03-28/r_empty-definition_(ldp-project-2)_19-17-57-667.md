error id: scala/Predef.Set#
file://<WORKSPACE>/src/main/scala/project_2/main.scala
empty definition using pc, found symbol in pc: 
found definition using semanticdb; symbol scala/Predef.Set#
empty definition using fallback
non-local guesses:

offset: 2540
uri: file://<WORKSPACE>/src/main/scala/project_2/main.scala
text:
```scala
package project_2

import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.functions._
import org.apache.spark.rdd._


object main{

  val seed = new java.util.Date().hashCode;
  val rand = new scala.util.Random(seed);

  class hash_function(numBuckets_in: Long) extends Serializable {  // a 2-universal hash family, numBuckets_in is the numer of buckets
    val p: Long = 2147483587;  // p is a prime around 2^31 so the computation will fit into 2^63 (Long)
    val a: Long = (rand.nextLong %(p-1)) + 1  // a is a random number is [1,p]
    val b: Long = (rand.nextLong % p) // b is a random number in [0,p]
    val numBuckets: Long = numBuckets_in

    def convert(s: String, ind: Int): Long = {
      if(ind==0)
        return 0;
      return (s(ind-1).toLong + 256 * (convert(s,ind-1))) % p;
    }

    def hash(s: String): Long = {
      return ((a * convert(s,s.length) + b) % p) % numBuckets;
    }

    def hash(t: Long): Long = {
      return ((a * t + b) % p) % numBuckets;
    }

    def zeroes(num: Long, remain: Long): Int =
    {
      if((num & 1) == 1 || remain==1)
        return 0;
      return 1+zeroes(num >> 1, remain >> 1);
    }

    def zeroes(num: Long): Int =        /*calculates #consecutive trialing zeroes  */
    {
      return zeroes(num, numBuckets)
    }
  }

  class four_universal_Radamacher_hash_function extends hash_function(2) {  // a 4-universal hash family, numBuckets_in is the numer of buckets
    override val a: Long = (rand.nextLong % p)   // a is a random number is [0,p]
    override val b: Long = (rand.nextLong % p) // b is a random number in [0,p]
    val c: Long = (rand.nextLong % p)   // c is a random number is [0,p]
    val d: Long = (rand.nextLong % p) // d is a random number in [0,p]

    override def hash(s: String): Long = {     /* returns +1 or -1 with prob. 1/2 */
      val t= convert(s,s.length)
      val t2 = t*t % p
      val t3 = t2*t % p
      return if ( ( ((a * t3 + b* t2 + c*t + d) % p) & 1) == 0 ) 1 else -1;
    }

    override def hash(t: Long): Long = {       /* returns +1 or -1 with prob. 1/2 */
      val t2 = t*t % p
      val t3 = t2*t % p
      return if( ( ((a * t3 + b* t2 + c*t + d) % p) & 1) == 0 ) 1 else -1;
    }
  }

  class BJKSTSketch(bucket_in: Set[(String, Int)] ,  z_in: Int, bucket_size_in: Int) extends Serializable {
/* A constructor that requies intialize the bucket and the z value. The bucket size is the bucket size of the sketch. */

    var bucket: Set@@[(String, Int)] = bucket_in
    var z: Int = z_in

    val BJKST_bucket_size = bucket_size_in

    def this(s: String, z_of_s: Int, bucket_size_in: Int){
      /* A constructor that allows you pass in a single string, zeroes of the string, and the bucket size to initialize the sketch */
      this(Set((s, z_of_s )) , z_of_s, bucket_size_in)
    }

    // Part 1
    def +(that: BJKSTSketch): BJKSTSketch = {    /* Merging two sketches */
      val merged = this.bucket union that.bucket
      val new_z = math.min(this.z, that.z)
      val trimmed = merged.toSeq.sortBy(_._2).take(BJKST_bucket_size).toSet
      new BJKSTSketch(trimmed, new_z, BJKST_bucket_size)
    }

    // Part 2
    def add_string(s: String, z_of_s: Int): BJKSTSketch = {   /* add a string to the sketch */
      if (z_of_s > z) return this
      val new_bucket = bucket + ((s, z_of_s))
      val trimmed = if (new_bucket.size > BJKST_bucket_size) {
        new_bucket.toSeq.sortBy(_._2).take(BJKST_bucket_size).toSet
      } else {
        new_bucket
      }
      val new_z = trimmed.map(_._2).min
      new BJKSTSketch(trimmed, new_z, BJKST_bucket_size)
    }
  }


  def tidemark(x: RDD[String], trials: Int): Double = {
    val h = Seq.fill(trials)(new hash_function(2000000000))

    def param0 = (accu1: Seq[Int], accu2: Seq[Int]) => Seq.range(0,trials).map(i => scala.math.max(accu1(i), accu2(i)))
    def param1 = (accu1: Seq[Int], s: String) => Seq.range(0,trials).map( i =>  scala.math.max(accu1(i), h(i).zeroes(h(i).hash(s))) )

    val x3 = x.aggregate(Seq.fill(trials)(0))( param1, param0)
    val ans = x3.map(z => scala.math.pow(2,0.5 + z)).sortWith(_ < _)( trials/2) /* Take the median of the trials */

    return ans
  }

  // Part 3
  def BJKST(x: RDD[String], width: Int, trials: Int) : Double = {
    val h = Seq.fill(trials)(new hash_function(1L << 20))
    val sketches = Seq.fill(trials)(new BJKSTSketch(Set.empty[(String, Int)], Int.MaxValue, width))
  
    val mergedSketches = x.aggregate(sketches)(
      (acc, s) => {
        acc.indices.map { i =>
          val z = h(i).zeroes(h(i).hash(s))
          acc(i).add_string(s, z)
        }
      },
      (acc1, acc2) => {
        acc1.indices.map { i =>
          acc1(i) + acc2(i)
        }
      }
    )
  
    val estimates = mergedSketches.map(sk => (sk.bucket.size - 1) * math.pow(2, sk.z))
    val sorted = estimates.sorted
    val median = sorted(trials / 2)
  
    return median
  }

  // Part 4
  def Tug_of_War(x: RDD[String], width: Int, depth:Int) : Long = {

  val totalSketches = width * depth
  val hashes = Seq.fill(totalSketches)(new four_universal_Radamacher_hash_function)

  val frequencyRDD = x.map(s => (s, 1L)).reduceByKey(_ + _)

  val allSketches = hashes.map { h =>
    val sum = frequencyRDD.map { case (s, count) =>
      h.hash(s) * count
    }.reduce(_ + _)
    sum * sum
  }

  val grouped = allSketches.grouped(width).toSeq
  val means = grouped.map(group => group.sum.toDouble / width)
  val sortedMeans = means.sorted
  val median = if (depth % 2 == 1) sortedMeans(depth / 2)
    else (sortedMeans(depth / 2 - 1) + sortedMeans(depth / 2)) / 2

  median.toLong
  }


  def exact_F0(x: RDD[String]) : Long = {
    val ans = x.distinct.count
    return ans
  }

  // Part 5
  def exact_F2(x: RDD[String]) : Long = {
    val ans = x.map(word => (word, 1))
    .reduceByKey(_ + _)
    .map { case (_, count) => count.toLong * count }
    .sum()
    .toLong
    return ans
  }



  def main(args: Array[String]) {
    val spark = SparkSession.builder().appName("Project_2").getOrCreate()

    if(args.length < 2) {
      println("Usage: project_2 input_path option = {BJKST, tidemark, ToW, exactF2, exactF0} ")
      sys.exit(1)
    }
    val input_path = args(0)

  //    val df = spark.read.format("csv").load("data/2014to2017.csv")
    val df = spark.read.format("csv").load(input_path)
    val dfrdd = df.rdd.map(row => row.getString(0))

    val startTimeMillis = System.currentTimeMillis()

    if(args(1)=="BJKST") {
      if (args.length != 4) {
        println("Usage: project_2 input_path BJKST #buckets trials")
        sys.exit(1)
      }
      val ans = BJKST(dfrdd, args(2).toInt, args(3).toInt)

      val endTimeMillis = System.currentTimeMillis()
      val durationSeconds = (endTimeMillis - startTimeMillis) / 1000

      println("==================================")
      println("BJKST Algorithm. Bucket Size:"+ args(2) + ". Trials:" + args(3) +". Time elapsed:" + durationSeconds + "s. Estimate: "+ans)
      println("==================================")
    }
    else if(args(1)=="tidemark") {
      if(args.length != 3) {
        println("Usage: project_2 input_path tidemark trials")
        sys.exit(1)
      }
      val ans = tidemark(dfrdd, args(2).toInt)
      val endTimeMillis = System.currentTimeMillis()
      val durationSeconds = (endTimeMillis - startTimeMillis) / 1000

      println("==================================")
      println("Tidemark Algorithm. Trials:" + args(2) + ". Time elapsed:" + durationSeconds + "s. Estimate: "+ans)
      println("==================================")

    }
    else if(args(1)=="ToW") {
       if(args.length != 4) {
         println("Usage: project_2 input_path ToW width depth")
         sys.exit(1)
      }
      val ans = Tug_of_War(dfrdd, args(2).toInt, args(3).toInt)
      val endTimeMillis = System.currentTimeMillis()
      val durationSeconds = (endTimeMillis - startTimeMillis) / 1000
      println("==================================")
      println("Tug-of-War F2 Approximation. Width :" +  args(2) + ". Depth: "+ args(3) + ". Time elapsed:" + durationSeconds + "s. Estimate: "+ans)
      println("==================================")
    }
    else if(args(1)=="exactF2") {
      if(args.length != 2) {
        println("Usage: project_2 input_path exactF2")
        sys.exit(1)
      }
      val ans = exact_F2(dfrdd)
      val endTimeMillis = System.currentTimeMillis()
      val durationSeconds = (endTimeMillis - startTimeMillis) / 1000

      println("==================================")
      println("Exact F2. Time elapsed:" + durationSeconds + "s. Estimate: "+ans)
      println("==================================")
    }
    else if(args(1)=="exactF0") {
      if(args.length != 2) {
        println("Usage: project_2 input_path exactF0")
        sys.exit(1)
      }
      val ans = exact_F0(dfrdd)
      val endTimeMillis = System.currentTimeMillis()
      val durationSeconds = (endTimeMillis - startTimeMillis) / 1000

      println("==================================")
      println("Exact F0. Time elapsed:" + durationSeconds + "s. Estimate: "+ans)
      println("==================================")
    }
    else {
      println("Usage: project_2 input_path option = {BJKST, tidemark, ToW, exactF2, exactF0} ")
      sys.exit(1)
    }

  }
}


```


#### Short summary: 

empty definition using pc, found symbol in pc: 