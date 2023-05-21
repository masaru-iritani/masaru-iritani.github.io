---
title: 無効な参照を経由した std::function 代入時のアクセス違反例外
categories: cpp
---

とあるクラスの `std::function` 型メンバー変数に値を代入しようとすると、`std::function` の内部で例外が発生して、暫く悩んでいました。

> Exception thrown: read access violation.
> **this** was 0xFFFFFFFFFFFFFFC7.
> ```
> > std::_Func_class<void,wchar_t const *>::_Getimpl() Line 1000
> > std::_Func_class<void,wchar_t const *>::_Empty() Line 894
> > std::_Func_class<void,wchar_t const *>::_Tidy() Line 957
> > std::function<void __cdecl(wchar_t const *)>::operator=(std::function<void __cdecl(wchar_t const *)> && _Right)
> > A::SetErrorHandler(std::function<void __cdecl(wchar_t const *)> handler) // 💥
> > A::X(...) // ❌
> ```

再現コードを抜粋します。💥 と ❌ は上記コールスタックと対応します。

```cpp
void A::SetErrorHandler(std::function<void(PCWSTR)> handler)
{
    m_errorHandler = std::move(handler); // 💥
}

void A::X(...)
{
    auto v = std::vector<std::unique_ptr<A>>{};
    auto& a1 = v.emplace_back(std::make_unique<A>(...));
    auto& a2 = v.emplace_back(std::make_unique<A>(...));
    m_v.push_back(std::move(v));

    auto errorHandler = [...](PCWSTR message){ ... };
    a1->SetErrorHandler(errorHandler); // ❌
    a2->SetErrorHandler(errorHandler);
}
```

このコードには 2 つの問題点が有ります。

1. `v.emplace_back` によって領域の再確保が行われると、`v` が所有する `std::unique_ptr` が再配置される。
1. `std::move` によって `std:unique_ptr` が `v` から `m_v` へ移動する。

どちらも `v.emplace_back` によって返された参照を無効に（つまり dangling reference に）してしまいます。

少し不格好ですが、変更箇所の大きさや他のコードとの統一性を考えて、最後の 2 行を次のように修正しました。

```cpp
m_v.back()[0]->SetErrorHandler(errorHandler);
m_v.back()[1]->SetErrorHandler(errorHandler);
```
