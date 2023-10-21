+++
title = "通过例子学习 Clojure"
date = "2019-05-13T15:22:05+08:00"
categories = [ "编程",]
tags = [ "code", "clojure",]
toc = "true"
+++


这份笔记试图打造最强的 clojure 小抄，keep refactoring it...

<!--more-->

## clojure 入门
```clojure
(ns clj-notes.core
  (:gen-class))
;:gen-class generate java class file

;Parameter is variable in the declaration of function.
;Argument is the actual value of this variable that gets passed to function.
;
;install leiningen:
;put lein.bat in your PATH
;open cmder,run: lein repl
;start repl,use exit,(exit),(quit) or ctrl+d to quit repl
(println "hello clojure")

;Symbols are used to bind names to values
;' will prevent a form from being evaluated
;'() same as (quote ())

;def global variable
;let local variable binding
(def object "light")
(println object)

(let [x 10
      y 20
      z 30]
  (+ x y z))
;=> 60

;data collection
;seq is abstract for list vector array
;map
(def dict {:k1 "v1" :k2 "v2"})
;keyword as function
(:k1 dict)                                                  ;return v1
;map as function
(dict :k1)                                                  ;return v1
(let [v (dict :k1)]
  (println v))

;also you can use get on seq or map
(get {:a 1 :b 2} :b)
;=> 2
;clojure.core/seq is a function that produces a sequence over the given argument. 
;Data types that clojure.core/seq can produce a sequence over are called seqable:
;
;Clojure collections
;Java maps
;All iterable types (types that implement java.util.Iterable)
;Java collections (java.util.Set, java.util.List, etc)
;Java arrays
;All types that implement java.lang.CharSequence interface, including Java strings
;All types that implement clojure.lang.Seqable interface
;nil

;function for seq or collection
;=
;count
;conj
;empty
;seq
;first
;rest
;next
;count
;counted?
;conj
;get
;assoc


;defn 定义函数
;defn- 定义ns内私有函数
(defn f
  "the second line is doc-string"
  {:added  "1.2"                                            ;this is attr-map
   :static true}
  [param]
  (print "hello " param))

(meta (var f))
;#' is the reader macro for var and works the exactly same
(meta #'f)

;fn create a function
(def f (fn [] (println "this is from fn function")))
;#() is the shortcut for fn
(def plus-one #(+ 1 %))
;% will be replaced with arguments passed to the function
;%1 is for the first argument, %2 is for the second and so on

(defn des
  [{k1 :k1}]                                                ;get :k1 value from argument (map) and binding it to k1(parameter)
  (println "destructing in map" k1))

(des dict)                                                  ;destructing in map v1
;key don't have to be keyword
(defn currency-of
  [{currency "currency"}]
  currency)
(defn currency-of
  [{currency 'currency}]
  currency)
;if want to destructing multi key,use :keys, in this case,parameter name(currency amount) 
;must same as arguments's keys(:currency :amount),can not use string as key
(defn currency-of
  [{:keys [currency amount]}]
  (* currency amount))

(currency-of {:currency "RMB" :amount 100000})              ;ok
(currency-of {"currency" "RMB" "amount" 100000})            ;currency will be nil,you will need use :strs or syms

(defn currency-strs
  [{:strs [currency amount]}]
  currency)
(currency-strs {"currency" "RMB" "amount" 100000})          ;ok


(defn currency-syms
  [{:syms [currency amount]}]
  currency)
(currency-syms {'currency "CNY" 'amount 100000})            ;ok

;use :or to give a default value for parameter
(defn currency-or
  [{:keys [currency amount] :or {currency "USD"}}]
  currency)
(currency-or {:amount 100000})                              ;=> "USD"

;use & for Variadic Functions parameters
(defn log
  [message & args]
  (println "args: " args))

;named params , achieved by Variadic Functions destructing
(defn job-info
  [& {:keys [name job income] :or {job "unemployed" income "$0.00"}}]
  (if name
    [name job income]
    (println "No name specified")))

;cation! arguments to job-info is not a map
(job-info :name "Robert" :job "Engineer")
;["Robert" "Engineer" "$0.00"]
;Without the use of a variadic argument list, 
;you would have to call the function with a single map argument such as
(job-info {:name "Robert" :job "Engineer"})

;destructuring example
;https://gist.github.com/john2x/e1dca953548bfdfb9844
(def my-vec [1 2 3])

(let [[a b c d] my-vec]
  (println a b c d))
;1 2 3 nil
(let [[a b & the-rest] my-vec]
  (println "a=" a "b=" b "the-rest=" the-rest))
;a= 1 b= 2 the-rest= (3)
(let [[:as all] my-vec]
  (println all))
;[1 2 3]
(let [[a :as all] my-vec]
  (println a all))
;1 [1 2 3]
(let [[a b & the-rest :as all] my-vec]
  (println a b the-rest all))
;1 2 (3) [1 2 3]
;note: & the-rest convert vector to list,
;but :as preserves them (as a list, or as a vector)
(def my-vec ["first" "second"])
(let [{a 0 b 1} my-vec]
  (println a b))                                            ;=> "first second"

;optional arguments to functions
(defn foo [a b & more-args]
  (println a b more-args))
(foo :a :b)                                                 ;; => :a :b nil
(foo :a :b :x)                                              ;; => :a :b (:x)
(foo :a :b :x :y :z)                                        ;; => :a :b (:x :y :z)

;map destructuring
(def my-hashmap {:a "A" :b "B" :c "C" :d "D"})
(def my-nested-hashmap {:a "A" :b "B" :c "C" :d "D" :q {:x "X" :y "Y" :z "Z"}})

(let [{a :a d :d} my-hashmap]
  (println a d))
;; => A D

(let [{a :a, b :b, {x :x, y :y} :q} my-nested-hashmap]
  (println a b x y))
;; => A B X Y
(let [{a :a, b :b, not-found :not-found, :or {not-found ":)"}, :as all} my-hashmap]
  (println a b not-found all))
;; => A B :) {:a A :b B :c C :d D}

;!!! There is no & rest for maps.



;everything but false and nil evaluates to true in Clojure.

;:as bind entire map to param
;See https://github.com/ring-clojure/ring/wiki/File-Uploads for explanation
(defn file-handler
  ;表示入参是一个map,里面有:params这个key,将:params
  ;[{{{tempfile :tempfile filename :filename} "file"} :params :as request}]
  [{{{tempfile :tempfile filename :filename} "file"} :params :as request}]
  (println request)
  (let [n (num-lines tempfile)]
    (response (str "File " filename " has " n " lines "))))

;a simple example
(defn first-first
  [[[i _] _]]
  i)

(first-first [[1 2] [3 4]])
;return 1

;(defn name doc-string? attr-map? [params*] prepost-map? body)
;(defn name doc-string? attr-map? ([params*] prepost-map? body) + attr-map?)
;function can have params type hint
(defn round
  "^double here is type hint"
  [^double d ^long precision]
  (let [factor (Math/pow 10 precision)]
    (/ (Math/floor (* d factor)) factor)))

;不定长参数
;重载函数
(defn bar
  ([a b] (bar a b 100))
  ([a b c] (* a b c)))

(bar 5 6)
(bar 5 6 3)


(defn keyworded-map [& {:keys [function sequence]}]
  (map function sequence))

(keyworded-map :sequence [1 2 3] :function #(+ % 2))

;trampoline
;trampoline


;namespace
;create-ns create a namespace
(create-ns 'zhi.moe.clj)

;in-ns move to a namespace
;require loads a namespace and
;refer refers the namespace.
;To do these at once, you can use use
(require 'clojure.by.example)
(clojure.by.example/favorite-language)
(use 'clojure.by.example)
;you can rename namespace
(require '[clojure.by.example :as temp-ns])

;ns macro creates a new namespace and gives you an opportunity to load other namespaces at the creation time

;import java class
(import java.util.Date)
(println (str  (new Date)))
;Wed Jul 24 22:55:24 CST 2019

;boolean
;In Clojure, everything except false and nil are true.
(if 1
  (println "it is true")
  (println "will never print"))

;if
(if true
  (println "executed when true")
  (println "executed when false"))

;use do to execute multi expressions
(if true
  (do
    (println "one")
    (println "two")))

;if-let:
(defn positive-number [numbers]
  (if-let [pos-nums (not-empty (filter pos? numbers))]
    pos-nums
    "no positive numbers"))

;when when-let case cond condp
;
(defn cond-test
  [n]
  (cond
    (= n 1) "n is 1"
    (and (> n 3) (< n 10)) "n is over 3 and under 10"
    :else "n is other"))

(cond-test 1000)


;string
(let [first "Hirokuni"
      last "Kim"]
  (str "My name is " first " " last))

;format
(format "My name is %s %s" "Hirokuni" "Kim")

;power function
(defn power
  [x n]
  (reduce * (repeat n x)))

;bigint,N is a literal for bigint
(+ 9223372036854775807 10N)

;list conj nth count
'(1 2 3)
;vector conj nth count .indexOf
[1 2 3]
(.indexOf [1 2 3] 4)

(count [1 2])

;set conj nth count disj sort contains? subset? superset?
#{1 2 3}

;map assoc merge keys vals
(let [os {:Apple "Mac" :Microsoft "Windows"}]
  (get os :Apple))

(assoc {:Apple "Mac" :Microsoft "Windows"} :Commodore "Amiga")

;Sequences are data types that abstract all more concrete data types with unified functions. 
;These functions are called the Seq library in Clojure.
;seq first rest cons concat map reduce into
;To add an element to the head of sequence, use cons.
(cons 4 [1 2 3])
(into [] `(1 2 3))
(reverse [1 2 3])
;get a sequence of infinite integers with iterate. Be careful, 
;though. Running this example will freeze your terminal since the evaluation of this expression never returns.
(doc iterate)

(doc range)
(repeatedly 5 (fn [] (println "hi!")))
;for each
(doseq [animal ["cat" "dog" "horse"]] (println animal))

(take 5 (range 0 100))
(take-while neg? [-3 -2 -1 0 1 2 3])
;drop will remove the first n elements
(drop 5 (range 0 10))
(drop-while neg? [-3 -2 -1 0 1 2 3])
;(0 1 2 3)

(remove pos? [-1 -2 3 4])
;(-1 -2)

(filter pos? [-1 2 3])
(partition-by #(< 3 %) [1 2 3 4 5 6])
(group-by #(< 3 %) [1 2 3 4 5 6 1 2 3])
(println (take 5 (iterate inc 0)))

;for compression
(for [x '(1 2 3)]
  (+ 10 x))

(doc for)
;双重for 循环
(for [x (range 10)
      y (range 20)
      :while (< y x)]
  [x y])

;<==> {x | x >0}
(for [x '(-1 1 2)
      :when (> x 0)]
  x)

(for [x [0 1 2 3 4 5]
      :let [y (* x 3)]
      :when (even? y)]
  y)

;meta data for function parameters
(defn round
  [^double d ^long precision]
  (let [factor (Math/pow 10 precision)]
    (/ (Math/floor (* d factor)) factor)))

;# is Dispatch character that tells the Clojure reader how to interpret the next character using a read table
;set
#{1 2 3}
;discard
{:a 1, #_#_:b 2, :c 3}
;regular expression
(re-matches #"^test$" "test")
;anonymous function
#(println %)
;var quote
(read-string "#'foo")
;symbolic values
(/ 1.0 0.0)                                                 ;##Inf
;tagged literals
(type #inst "2014-05-19T19:12:37.925-00:00")                ;java.util.Date
;meta
(meta #'fn-name)
;reader conditionals 
#?(:clj     (Clojure expression)
   :cljs    (ClojureScript expression)
   :cljr    (Clojure CLR expression)
   :default (fallthrough expression))
;#?@ splicing reader conditional
(defn build-list []
  (list #?@(:clj  [5 6 7 8]
            :cljs [1 2 3 4])))                              ;return [5 6 7 8] when run on clojure
;#= allows the reader to evaluate an arbitrary form during read time
(read-string "#=(+ 3 4)")                                   ;7


;Recursion
;simple recursion
(defn fibo
  "this is recursion function"
  [n]
  (if (or (= n 0) (= n 1))
    n
    (+ (fibo (- n 1)) (fibo (- n 2)))))
;do not do this!!! take a long time to finish
(fibo 1000)

;use recur
(defn fibo-recur [iteration]
  (let [fibo (fn [one two n]
               (if (= iteration n)
                 one
                 (recur two (+ one two) (inc n))))]
    ;recur re-binds it's arguments to new values and call the function with the new values
    ;fibo is an inner function
    (fibo 0N 1N 0)))

(fibo-recur 1000)
;it is really fast
;notes
;with simple recursion, each recursive call creates a stack frame which is 
;a data to store the information of the called function on memory.
;Doing deep recursion requires large memory for stack frames, but since it cannot, 
;we get StackOverflowError
;尾递归
;A function is tail recursive when the recursion is happening at the end of it's definition
;In other words, a tail recursive function must return itself as it's returned value.
;When you use recur, it makes sure you are doing tail recursion

(doc loop)
;loop/recur is merely a friendly way to write recursion code.
;All imperative loops can be converted to recursions and all recursions can be converted to loops,
;so Clojure chose recursions.
;Although you can write code that looks like an imperative loop with loop/recur,
;Clojure is doing recursion under the hood.

;
(defmacro unless [test then]
  "Evaluates then when test evaluates to be falsey"
  (list 'if (list 'not test)
        then))

(macroexpand '(unless false (println "hi")))
;' quoting
;` syntax-quoting returns the fully qualified namespace.
;Using fully qualified namespace is very important in order to avoid name conflicts when defining macro.
;~ unquote

`(+ ~(list 1 2 3))
;(clojure.core/+ (1 2 3))

`(+ ~@(list 1 2 3))
;(clojure.core/+ 1 2 3)
;The ~@ unquote splice works just like ~ unquote,
;except it expands a sequence and splice the contents of 
;the sequence into the enclosing syntax-quoted data structure

;thread first macro
(-> []
    (conj 1)
    (conj 2)
    (conj 3))
;[1 2 3]

(first (.split (.replace (.toUpperCase "a b c d") "A" "X") " "))
;"X"

;;Perhaps easier to read:
;-> 后面是初始参数,第2行开始每一行是一个函数调用,
;且上一行的返回值会作为这一行第一个参数(这就是thread first)的first含义
;这里的thread是管道的意思,而不是并发编程的线程
;如果省略(),那么野生符号(bare symbol)和keyword都会当作一个函数调用,
;例如,这里的.toUpperCase是bare symbol,等效于(.toUpperCase ,,,)
;clojure中 逗号等于空白符,所以上面用,,,表示将会插入的参数(即"a b c d")
(-> "a b c d"
    .toUpperCase
    (.replace "A" "X")
    (.split " ")
    first)
;same as follow, ,,, is equals whitespace
(-> "a b c d"
    (.toUpperCase,,,)
    (.replace "A" "X")
    (.split " ")
    first)

;suppose a function
(defn calculate []
  (reduce + (map #(* % %) (filter odd? (range 10)))))

;same as
;上一行的结果作为最后一个参数插入,这叫thread last
(defn calculate* []
  (->> (range 10)
       (filter odd?,,,)
       (map #(* % %),,,)
       (reduce +,,,)))

;如果想要指定每次插入的位置那么需要用 as->
;v是每一行的返回值的名称,这样你可以在下一行任意参数位置指定
(as-> [:foo :bar] v
      (map name v)
      (first v)
      (.substring v 1))


;
;destructing
({:keys [firstname lastname] :as person} {:firstname "John" :lastname "Smith"})


;future and deref
(let [future-val (future (inc 1))]
  (println (deref future-val)))
;deref == @
(let [future-val (future (inc 1))]
  (println @future-val))

(def my-future (future (Thread/sleep 5000)))
(repeatedly 6
            (fn []
              (println (realized? my-future))
              (Thread/sleep 1000)))

(doc future)


;promise
(def my-promise (promise))
;you define a promise
(def listen-and-callback (fn []
                           (println "Start listening...")
                           (future (println "Callback fired: " @my-promise))))

(defn do-time-consuming-job []
  (Thread/sleep 5000)
  (deliver my-promise "delivered value"))

(listen-and-callback)
(do-time-consuming-job)

;atom is like mutable var in other languages but atom is thread safe

;ref dosync ref-set alter
(def my-ref (ref 0))
(dosync
  (alter my-ref
         (fn [current_ref]
           (inc current_ref))))

(print @my-ref)

(def user (ref {}))
(dosync
  (alter user merge {:name "Kim"})
  (throw (Exception. "something wrong happens!"))
  (alter user merge {:age 32}))

(def user-record (atom {}))

(do (swap! user-record merge {:name "Kim"})
    (throw (Exception. "something wrong happens!"))
    (swap! user-record merge {:age 32}))


;Java
(new java.util.Date "2016/2/19")
(java.util.Date.)
(java.util.Date. "2016/2/19")
(Math/pow 2 3)                                              ;static method
(def rnd (new java.util.Random))
(. rnd nextInt 10)

(let [date1 (new java.util.Date)
      date2 (new java.util.Date)]
  (.equals date1 date2))

;(.instanceMember instance args*)
;(.instanceMember Classname args*)
;(.-instanceField instance)
;(Classname/staticMethod args*)
;Classname/staticField

;;;

(defn geohash [lat lng]
  (println "geohash:" lat lng)
  ;;this function take two separate values as params.
  ;;and it return a geohash for that position
  )

(let [{:strs [lat lng] :as coord} {"lat" 51.503331, "lng" -0.119500}]
  (println "calculating geohash for coordinates: " coord)
  (geohash lat lng))


;assoc-in associate使加入


```