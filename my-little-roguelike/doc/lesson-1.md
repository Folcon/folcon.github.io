# Draw something on the "screen"
```clojure
(def font-scale 20)

(defn precreated-game-view [width height entity]
  [:svg {:width  width
         :height height
         :on-context-menu #(identity false)
         :view-box (str 0 " " 0 " " width " " height)
         :pointer-events :all}
   (let [{:keys [x y glyph type]} entity]
     [type {:x (+ (* font-scale x) 10) :y (+ (* font-scale y) 10)} glyph])])

(defn game-view []
  (let [entity {:x 5 :y 4 :glyph "@" :type :text}]
    [precreated-game-view 200 200 entity]))
```

# Pressing characters prints stuff? (Not sure if this step is necessary)
```clojure
(defn app []
  (let [on-key-press (fn [e]
                       (.log js/console "key press" (.-charCode e))
                       (if (= 13 (.-charCode e))
                         (println "ENTER")
                         (println "NOT ENTER")))]
    (.removeEventListener (.-body js/document) "keypress" on-key-press)
    (.addEventListener (.-body js/document) "keypress" on-key-press)
    [:div]))
```

# Pressing characters moves stuff
```clojure
(def font-scale 20)

(defn precreated-game-view [width height pieces]
  [:svg {:width  width
         :height height
         :on-context-menu #(identity false)
         :view-box (str 0 " " 0 " " width " " height)
         :pointer-events :all}
   (into [:g] (map (fn [p] (let [{:keys [x y glyph type]} p] [type {:x (+ (* font-scale x) 10) :y (+ (* font-scale y) 10)} glyph])) pieces))])

(defn game-view []
  [precreated-game-view 200 200 [{:x 5 :y 4 :glyph "@" :type :text}]])

(defn app []
  (let [key->kw {37 :left
                 38 :up
                 39 :right
                 40 :down
                 65 :left
                 87 :up
                 68 :right
                 83 :down}
        on-key-press (fn [e]
                       (.log js/console "key press" (.-which e) (get key->kw (.-which e)))
                       (if (= 13 (.-which e))
                         (println "ENTER")
                         (println "NOT ENTER")))]
    (.removeEventListener (.-body js/document) "keydown" on-key-press)
    (.addEventListener (.-body js/document) "keydown" on-key-press)
    [game-view]))
```

# Multiple Entities
```clojure
(def font-scale 20)

(defn precreated-game-view [width height pieces]
  [:svg {:width  width
         :height height
         :on-context-menu #(identity false)
         :view-box (str 0 " " 0 " " width " " height)
         :pointer-events :all}
   (into [:g] (map (fn [p] (let [{:keys [x y glyph type]} p] [type {:x (+ (* font-scale x) 10) :y (+ (* font-scale y) 10)} glyph])) pieces))])

(defn game-view []
  [precreated-game-view 200 200 [{:x 5 :y 4 :glyph "@" :type :text}]])

(defn app []
  (let [key->kw {37 :left
                 38 :up
                 39 :right
                 40 :down
                 65 :left
                 87 :up
                 68 :right
                 83 :down}
        on-key-press (fn [e]
                       (.log js/console "key press" (.-which e) (get key->kw (.-which e)))
                       (if (= 13 (.-which e))
                         (println "ENTER")
                         (println "NOT ENTER")))]
    (.removeEventListener (.-body js/document) "keydown" on-key-press)
    (.addEventListener (.-body js/document) "keydown" on-key-press)
    [game-view]))
```


# Complete Program with debug
```
(rf/reg-event-db
  :init
  (fn [db _]
    (assoc db :player {:x 1 :y 1 :glyph "@" :type :text})))

(rf/reg-sub
  :player
  (fn [db _]
    (get db :player)))

(rf/reg-event-db
  :move-player
  (fn [db [_ x y]]
    (-> db
      (update-in [:player :x] + x)
      (update-in [:player :y] + y)
      (#(do (.log js/console (pr-str %)) %)))))



(defn game-view [entity]
 (fn [entity]
   [:svg {:width  200
          :height 200
          :on-context-menu #(identity false)
          :view-box (str 0 " " 0 " " 200 " " 200)
          :pointer-events :all
          :style {:background-color :white}}
      (let [{:keys [x y glyph type]} @entity]
        [type {:x (+ (* 10 x) 10) :y (+ (* 10 y) 10)} glyph])]))

(defn screen-view []
  (let [entity (rf/subscribe [:player])]
    [game-view entity]))

(defn app []
  (let [key-ent->dir {37 :left
                      38 :up
                      39 :right
                      40 :down
                      65 :left
                      87 :up
                      68 :right
                      83 :down}
        dir->move {:left  [-1  0]
                   :right [ 1  0]
                   :up    [ 0 -1]
                   :down  [ 0  1]}

         on-key-press (fn [e]
                        (let [dir (key-ent->dir (.-which e))
                              move (dir->move dir)
                              [x y] move]
                          (when (and x y)
                            (.log js/console "a" (pr-str dir) x y (pr-str @(rf/subscribe [:player])))
                            (rf/dispatch [:move-player x y]))))]
    (.removeEventListener (.-body js/document) "keydown" on-key-press)
    (.addEventListener (.-body js/document) "keydown" on-key-press)
    (rf/dispatch [:init])
    [screen-view]))
```