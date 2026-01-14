### About the Pixel Editor project

This post will go over some code used in the pixel editor [paint project](/paint/index.html). This is a single page app written in Haskell with the miso library which allows for a model-view-controller(mvc) system for the web page via GHCJS (haskell to javascript transpiler). I will see if I can recreate my dev setup for this as it is kind of old; might be better to actually document it now so I won't forget again.

The mvc system needs to have a model, as it is central to the function of the app ( View <- Model <-> Controller ). The following will be the model's definition in Haskell.

```
type Grid =  M.Map Int JSString
type GridStore =  M.Map JSString Grid
data GridControl = FillSwitch | PaintSwitch | DragSwitch | NoopSwitch deriving (Eq, Show)-- changes actrions for clicking on the grid

-- | Type synonym for an application model
data Model = Model { mouseCoords :: (Int, Int) --mousetracking for color picker
                   , getArrows :: Arrows --arrows for index movement
                   , color :: JSString --selected color
                   , selected :: (Int,Int) --index on grid
                   , pix :: Int --how big to draw each pixel
                   , grid :: Grid --map from a pos. int to a color name or rgb value
                   , gridY :: YY JSString --current state of the grid
                   , gridCont :: GridControl --changes behavior of grid cells
                   
                   , store :: GridStore --map to store pictures along with titles
                   , title :: JSString --title to store current grid under
                   
                   }
           deriving (Show, Eq)
```

I tried documenting a little bit with these comments. Some are fairly obvious: `mouseCoords`, `color`, `selected`, `pix`, and `title` would be a few. The `gridY` and `getArrows` I will have to explain some more. 
I know I made this while deep in the weeds reading about comonads and zippers, more about that in a bit. But the main component is the grid which has the type `YY JSString`. What is that? 

```
data Y a = Y { _yl :: S.Seq a
             , _yc :: Maybe a
             , _yi :: Int
             } deriving (Eq, Show)

newtype YY a = YY { _unyy :: Y (Y a) }
  deriving (Eq,Show)

makeLenses ''Y
makeLenses ''YY
```

This is could have some better documentation. It would seem I need to do some housekeeping on this project once I figure everything out. But it looks like a grid: the `Y`'s `_yl` field forms a sequence of values and the newtype `YY` would be another sequence of those `Y`s effectively formed the `grid`. So what is `_yc` and `_yi`?
I believe more clues are found in the instances of comonad and zipper for this datatype `Y`:

```
instance Comonad Y where
  extract = maybe (error "cursor not on grid") id . _yc 
  duplicate y = Y (S.fromFunction (size y - 1) fn) (Just y) ( y ^. yi )
    where fn k = compose (k + 1) ( shift YL ) y

instance Zipper Y where
  type Index Y = Int
  data Direction Y = YL | YR deriving (Eq, Show)
  cursor = _yc
  index = _yi
  resize a y@(Y l c i) n = let s = size y in
    case compare s n of
      LT -> Y (l <> S.replicate (n - s) a) c i
      GT -> uncurry (Y (S.take n l)) $ if i < n then (c,i) else (Just a,n) 
      EQ -> y 
  size (Y l _ _) = S.length l 
  (!) (Y l c _) k = l `S.index` k
  adjust f k y@(Y l c i) = case l ?! k of
    Nothing  -> y
    (Just j) -> y { _yl = S.adjust f k l, _yc = if k == i then f <$> c else c  }
  toList (Y l c i) = foldr (:) [] l
  fromMap _ [] = error "Zipper must have length greater than zero."
  fromMap a m = Y (S.fromList ys) (Just . snd $ minimumBy (comparing fst) m) 0
    where ys = fmap snd m      
  shift d v@(Y l c i)
    | S.null l = v -- shifting length zero amounts to nothing
    | d == YL   = Y l ( l ?! (i - 1) ) (i - 1)
    | d == YR   = Y l ( l ?! (i + 1) ) (i + 1)
```

Quite a bit there so won't go through it all. But the mystery is solved: the `_yc` corresponds to `cursor` and `_yi` to `index`. A few of these other functions seems useful in the implementation for navigating the grid. For example `resize` is used when changing the dimensions of the grid:

```
resizeR t@(readMay.unpack -> Nothing :: Maybe Int) = Id
resizeR t@(readMay.unpack -> Just x :: Maybe Int)
  | 42 > x && x > 0 = RedrawGrid $ YY $ resize (Y (S.replicate e c) (Just c) 0)  (_unyy y) x
  | True = Id

resizeC t@(readMay.unpack -> Nothing :: Maybe Int) = Id
resizeC t@(readMay.unpack -> Just x :: Maybe Int)
  | 42 > x && x > 0 = RedrawGrid $ YY $ fmap (flip (resize c) x) (_unyy y)
  | True = Id
```

The type `RedrawGrid` is one of the actions used in miso. The actions would be what the controller is in mvc. Actions data are passed into an Effect monad. Not going to mention much more about that now since I am realizing that I should just work on moving this project into the new miso version, and probably cleaning up a bunch of unused code; that and documenting everything this time. But the main reason I wanted to share this project is an excuse to post my cat's picture. Puffin, you were a fantastic cat, I will always miss you!

![puf](/assets/img/about-paint/PIXELPUF.png)
![poofer bear](/assets/img/about-paint/PPF.png)

I hope to one day revisit this project and make some nice updates. 