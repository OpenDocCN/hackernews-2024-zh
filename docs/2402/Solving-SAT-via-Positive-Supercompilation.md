<!--yml
category: 未分类
date: 2024-05-27 14:29:26
-->

# Solving SAT via Positive Supercompilation

> 来源：[https://hirrolot.github.io/posts/sat-supercompilation.html](https://hirrolot.github.io/posts/sat-supercompilation.html)

```
 check
 ~expected:
 ( "g0(a, b, c, d, e, f, g, h, i, k, l, m, n, o, p)",
 [
 "g0(F(), b, c, d, e, f, g, h, i, k, l, m, n, o, p) = g1(b, c, d, e, \
 f, g, h, i, k, l, m, n, o, p)";
 "g0(T(), b, c, d, e, f, g, h, i, k, l, m, n, o, p) = g30(b, c, d, e, \
 f, g, h, i, k, l, m, n, o, p)";
 "g1(F(), c, d, e, f, g, h, i, k, l, m, n, o, p) = F()";
 "g1(T(), c, d, e, f, g, h, i, k, l, m, n, o, p) = g2(d, c, e, f, g, \
 h, i, k, l, m, n, o, p)";
 "g10(F(), h, i, k, l, m, n, o) = g11(h, i, k, l, m, n, o)";
 "g10(T(), h, i, k, l, m, n, o) = F()";
 "g11(F(), i, k, l, m, n, o) = g12(i)";
 "g11(T(), i, k, l, m, n, o) = g13(n, i, k, l, m, o)";
 "g12(F()) = F()";
 "g12(T()) = F()";
 "g13(F(), i, k, l, m, o) = g14(i, k, l, m, o)";
 "g13(T(), i, k, l, m, o) = F()";
 "g14(F(), k, l, m, o) = F()";
 "g14(T(), k, l, m, o) = g15(l, k, m, o)";
 "g15(F(), k, m, o) = F()";
 "g15(T(), k, m, o) = g16(k, m, o)";
 "g16(F(), m, o) = g17(m, o)";
 "g16(T(), m, o) = g19(m, o)";
 "g17(F(), o) = F()";
 "g17(T(), o) = g18(o)";
 "g18(F()) = F()";
 "g18(T()) = F()";
 "g19(F(), o) = F()";
 "g19(T(), o) = g20(o)";
 "g2(F(), c, e, f, g, h, i, k, l, m, n, o, p) = g3(c, e, f, g, h, i, \
 k, l, m, n, o, p)";
 "g2(T(), c, e, f, g, h, i, k, l, m, n, o, p) = F()";
 "g20(F()) = F()";
 "g20(T()) = F()";
 "g21(F(), i, k, l, m, n, o, p) = g22(i)";
 "g21(T(), i, k, l, m, n, o, p) = g23(n, i, k, l, m, o, p)";
 "g22(F()) = F()";
 "g22(T()) = F()";
 "g23(F(), i, k, l, m, o, p) = g24(i, k, l, m, o, p)";
 "g23(T(), i, k, l, m, o, p) = F()";
 "g24(F(), k, l, m, o, p) = F()";
 "g24(T(), k, l, m, o, p) = g25(k, l, m, o, p)";
 "g25(F(), l, m, o, p) = g26(l, m, o, p)";
 "g25(T(), l, m, o, p) = F()";
 "g26(F(), m, o, p) = F()";
 "g26(T(), m, o, p) = g27(m, o, p)";
 "g27(F(), o, p) = F()";
 "g27(T(), o, p) = g28(o, p)";
 "g28(F(), p) = g29(p)";
 "g28(T(), p) = F()";
 "g29(F()) = F()";
 "g29(T()) = T()";
 "g3(F(), e, f, g, h, i, k, l, m, n, o, p) = F()";
 "g3(T(), e, f, g, h, i, k, l, m, n, o, p) = g4(e, f, g, h, i, k, l, \
 m, n, o, p)";
 "g30(F(), c, d, e, f, g, h, i, k, l, m, n, o, p) = g31(c, d, e, f, \
 g, h, i, k, l, m, n, o, p)";
 "g30(T(), c, d, e, f, g, h, i, k, l, m, n, o, p) = g66(d, c, e, f, \
 g, h, i, k, l, m, n, o, p)";
 "g31(F(), d, e, f, g, h, i, k, l, m, n, o, p) = g32(d, e, f, g)";
 "g31(T(), d, e, f, g, h, i, k, l, m, n, o, p) = g36(d, e, f, g, h, \
 i, k, l, m, n, o, p)";
 "g32(F(), e, f, g) = F()";
 "g32(T(), e, f, g) = g33(e, f, g)";
 "g33(F(), f, g) = g34(f, g)";
 "g33(T(), f, g) = F()";
 "g34(F(), g) = g35(g)";
 "g34(T(), g) = F()";
 "g35(F()) = F()";
 "g35(T()) = F()";
 "g36(F(), e, f, g, h, i, k, l, m, n, o, p) = g37(e, f, g, h, i, k, \
 l, m, n, o, p)";
 "g36(T(), e, f, g, h, i, k, l, m, n, o, p) = g63(e, f, g)";
 "g37(F(), f, g, h, i, k, l, m, n, o, p) = g38(f, g)";
 "g37(T(), f, g, h, i, k, l, m, n, o, p) = g40(f, g, h, i, k, l, m, \
 n, o, p)";
 "g38(F(), g) = g39(g)";
 "g38(T(), g) = F()";
 "g39(F()) = F()";
 "g39(T()) = F()";
 "g4(F(), f, g, h, i, k, l, m, n, o, p) = g5(f, g)";
 "g4(T(), f, g, h, i, k, l, m, n, o, p) = g7(f, g, h, i, k, l, m, n, \
 o, p)";
 "g40(F(), g, h, i, k, l, m, n, o, p) = g41(g)";
 "g40(T(), g, h, i, k, l, m, n, o, p) = g42(g, h, i, k, l, m, n, o, p)";
 "g41(F()) = F()";
 "g41(T()) = F()";
 "g42(F(), h, i, k, l, m, n, o, p) = g43(p, h, i, k, l, m, n, o)";
 "g42(T(), h, i, k, l, m, n, o, p) = g54(h, i, k, l, m, n, o, p)";
 "g43(F(), h, i, k, l, m, n, o) = g44(h, i, k, l, m, n, o)";
 "g43(T(), h, i, k, l, m, n, o) = F()";
 "g44(F(), i, k, l, m, n, o) = g45(i)";
 "g44(T(), i, k, l, m, n, o) = g46(n, i, k, l, m, o)";
 "g45(F()) = F()";
 "g45(T()) = F()";
 "g46(F(), i, k, l, m, o) = g47(i, k, l, m, o)";
 "g46(T(), i, k, l, m, o) = F()";
 "g47(F(), k, l, m, o) = F()";
 "g47(T(), k, l, m, o) = g48(l, k, m, o)";
 "g48(F(), k, m, o) = F()";
 "g48(T(), k, m, o) = g49(k, m, o)";
 "g49(F(), m, o) = g50(m, o)";
 "g49(T(), m, o) = g52(m, o)";
 "g5(F(), g) = g6(g)";
 "g5(T(), g) = F()";
 "g50(F(), o) = F()";
 "g50(T(), o) = g51(o)";
 "g51(F()) = F()";
 "g51(T()) = F()";
 "g52(F(), o) = F()";
 "g52(T(), o) = g53(o)";
 "g53(F()) = F()";
 "g53(T()) = F()";
 "g54(F(), i, k, l, m, n, o, p) = g55(i)";
 "g54(T(), i, k, l, m, n, o, p) = g56(n, i, k, l, m, o, p)";
 "g55(F()) = F()";
 "g55(T()) = F()";
 "g56(F(), i, k, l, m, o, p) = g57(i, k, l, m, o, p)";
 "g56(T(), i, k, l, m, o, p) = F()";
 "g57(F(), k, l, m, o, p) = F()";
 "g57(T(), k, l, m, o, p) = g58(k, l, m, o, p)";
 "g58(F(), l, m, o, p) = g59(l, m, o, p)";
 "g58(T(), l, m, o, p) = F()";
 "g59(F(), m, o, p) = F()";
 "g59(T(), m, o, p) = g60(m, o, p)";
 "g6(F()) = F()";
 "g6(T()) = F()";
 "g60(F(), o, p) = F()";
 "g60(T(), o, p) = g61(o, p)";
 "g61(F(), p) = g62(p)";
 "g61(T(), p) = F()";
 "g62(F()) = F()";
 "g62(T()) = T()";
 "g63(F(), f, g) = g64(f, g)";
 "g63(T(), f, g) = F()";
 "g64(F(), g) = g65(g)";
 "g64(T(), g) = F()";
 "g65(F()) = F()";
 "g65(T()) = F()";
 "g66(F(), c, e, f, g, h, i, k, l, m, n, o, p) = g67(c, e, f, g, h, \
 i, k, l, m, n, o, p)";
 "g66(T(), c, e, f, g, h, i, k, l, m, n, o, p) = F()";
 "g67(F(), e, f, g, h, i, k, l, m, n, o, p) = F()";
 "g67(T(), e, f, g, h, i, k, l, m, n, o, p) = g68(e, f, g, h, i, k, \
 l, m, n, o, p)";
 "g68(F(), f, g, h, i, k, l, m, n, o, p) = g69(f, g)";
 "g68(T(), f, g, h, i, k, l, m, n, o, p) = g71(f, g, h, i, k, l, m, \
 n, o, p)";
 "g69(F(), g) = g70(g)";
 "g69(T(), g) = F()";
 "g7(F(), g, h, i, k, l, m, n, o, p) = g8(g)";
 "g7(T(), g, h, i, k, l, m, n, o, p) = g9(g, h, i, k, l, m, n, o, p)";
 "g70(F()) = F()";
 "g70(T()) = F()";
 "g71(F(), g, h, i, k, l, m, n, o, p) = g72(g)";
 "g71(T(), g, h, i, k, l, m, n, o, p) = g73(g, h, i, k, l, m, n, o, p)";
 "g72(F()) = F()";
 "g72(T()) = F()";
 "g73(F(), h, i, k, l, m, n, o, p) = g74(p, h, i, k, l, m, n, o)";
 "g73(T(), h, i, k, l, m, n, o, p) = g85(h, i, k, l, m, n, o, p)";
 "g74(F(), h, i, k, l, m, n, o) = g75(h, i, k, l, m, n, o)";
 "g74(T(), h, i, k, l, m, n, o) = F()";
 "g75(F(), i, k, l, m, n, o) = g76(i)";
 "g75(T(), i, k, l, m, n, o) = g77(n, i, k, l, m, o)";
 "g76(F()) = F()";
 "g76(T()) = F()";
 "g77(F(), i, k, l, m, o) = g78(i, k, l, m, o)";
 "g77(T(), i, k, l, m, o) = F()";
 "g78(F(), k, l, m, o) = F()";
 "g78(T(), k, l, m, o) = g79(l, k, m, o)";
 "g79(F(), k, m, o) = F()";
 "g79(T(), k, m, o) = g80(k, m, o)";
 "g8(F()) = F()";
 "g8(T()) = F()";
 "g80(F(), m, o) = g81(m, o)";
 "g80(T(), m, o) = g83(m, o)";
 "g81(F(), o) = F()";
 "g81(T(), o) = g82(o)";
 "g82(F()) = F()";
 "g82(T()) = F()";
 "g83(F(), o) = F()";
 "g83(T(), o) = g84(o)";
 "g84(F()) = F()";
 "g84(T()) = F()";
 "g85(F(), i, k, l, m, n, o, p) = g86(i)";
 "g85(T(), i, k, l, m, n, o, p) = g87(n, i, k, l, m, o, p)";
 "g86(F()) = F()";
 "g86(T()) = F()";
 "g87(F(), i, k, l, m, o, p) = g88(i, k, l, m, o, p)";
 "g87(T(), i, k, l, m, o, p) = F()";
 "g88(F(), k, l, m, o, p) = F()";
 "g88(T(), k, l, m, o, p) = g89(k, l, m, o, p)";
 "g89(F(), l, m, o, p) = g90(l, m, o, p)";
 "g89(T(), l, m, o, p) = F()";
 "g9(F(), h, i, k, l, m, n, o, p) = g10(p, h, i, k, l, m, n, o)";
 "g9(T(), h, i, k, l, m, n, o, p) = g21(h, i, k, l, m, n, o, p)";
 "g90(F(), m, o, p) = F()";
 "g90(T(), m, o, p) = g91(m, o, p)";
 "g91(F(), o, p) = F()";
 "g91(T(), o, p) = g92(o, p)";
 "g92(F(), p) = g93(p)";
 "g92(T(), p) = F()";
 "g93(F()) = F()";
 "g93(T()) = T()";
 ] )
 (formula
 [
 clause [ `Var "a"; `Var "b" ];
 clause [ `Not "b"; `Not "d" ];
 clause [ `Var "c"; `Var "d" ];
 clause [ `Not "d"; `Not "e" ];
 clause [ `Var "e"; `Not "f" ];
 clause [ `Var "f"; `Not "g" ];
 clause [ `Var "f"; `Var "g" ];
 clause [ `Var "g"; `Not "p" ];
 clause [ `Var "h"; `Not "i" ];
 clause [ `Not "h"; `Not "n" ];
 clause [ `Var "i"; `Var "g" ];
 clause [ `Var "i"; `Not "g" ];
 clause [ `Not "g"; `Not "k" ];
 clause [ `Var "g"; `Var "l" ];
 clause [ `Var "k"; `Var "l" ];
 clause [ `Var "m"; `Var "n" ];
 clause [ `Var "n"; `Not "o" ];
 clause [ `Var "o"; `Var "p" ];
 ]);
```