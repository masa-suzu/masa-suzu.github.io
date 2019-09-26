# Functor 型クラス

「すごいHaskellたのしく学ぼう」の7章より。

```Haskell
-- 「型 a から型 b への関数」と「型 a に適用された Functor 値」をとり、
-- 「型 b に適用された Functor 値」を返す。"箱っぽい"なにか。
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

```Haskell
-- リスト Functor
-- map :: (a -> b) -> [a] -> [b]
instance Functor [] where
    fmap = map

-- MayBe Functor
instance Functor MayBe where
    fmap f (Just x) = Just (f x)
    fmap f Nothing = Nothing

-- Either "a" Functor
-- 型 Either a b を「型 b の入った箱」か「エラー型 a の箱」と考え、
-- 型 a を部分適用(固定)する。
instance Functor (Either a) where
    fmap f (Right x) = Right (f x)
    fmap f (Left x) = Left x

```
