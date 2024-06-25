<!--yml

category: 未分类

date: 2024-05-27 14:31:11

-->

# GMD - 比较 Julia 在 CPU 与 GPU 上的性能以及 Julia-MPI 与 Fortran-MPI：MPAS-Ocean 的案例研究（版本 7.1）

> 来源：[`gmd.copernicus.org/articles/16/5539/2023/`](https://gmd.copernicus.org/articles/16/5539/2023/)

Besard, T., Foket, C., and De Sutter, B.: 有效可扩展编程：在 GPU 上释放 Julia 的潜能，IEEE T. Parall. Distr., 30, 827–841, [`doi.org/10.1109/TPDS.2018.2872064`](https://doi.org/10.1109/TPDS.2018.2872064), 2018. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.19)

Besard, T., Churavy, V., Edelman, A., and De Sutter, B.: 异构和分布式平台的快速软件原型设计，Adv. Eng. Softw., 132, 29–46, 2019. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.19)

Bezanson, J., Edelman, A., Karpinski, S., and Shah, V. B.: Julia：一种全新的数值计算方法，SIAM 评论，59，65–98，2017. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.11)

Bishnu, S.: 偏微分方程和海洋模型的时间步进方法，Zenodo，[`doi.org/10.5281/zenodo.7439539`](https://doi.org/10.5281/zenodo.7439539), 2021. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_text.29), [b](https://gmd.copernicus.org/articles/16/5539/2023/#xref_text.38), [c](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.39)

Bishnu, S.: 旋转浅水验证套件，Zenodo [code], [`doi.org/10.5281/zenodo.7421135`](https://doi.org/10.5281/zenodo.7421135), 2022. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.26), [b](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.42), [c](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.45), [d](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.46)

Bishnu, S., Petersen, M., Quaife, B., and Schoonover, J.: 海洋模型条转换器的验证套件测试案例，正在审阅中，[`doi.org/10.22541/essoar.167100170.03833124/v1`](https://doi.org/10.22541/essoar.167100170.03833124/v1), 2022. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_text.28), [b](https://gmd.copernicus.org/articles/16/5539/2023/#xref_text.37)

Bleichrodt, F., Bisseling, R. H., and Dijkstra, H. A.: 使用 GPU 加速条转换器的海洋模型，Ocean Model., 41, 16–21, [`doi.org/10.1016/j.ocemod.2011.10.001`](https://doi.org/10.1016/j.ocemod.2011.10.001), 2012. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.4)

Byrne, S., Wilcox, L. C., and Churavy, V.: MPI. jl：消息传递接口的 Julia 绑定，在：JuliaCon Conferences 论文集，1，68，[`doi.org/10.21105/jcon.00068`](https://doi.org/10.21105/jcon.00068), 2021. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.20)

Caldwell, P. M., Mametjanov, A., Tang, Q., Van Roekel, L. P., Golaz, J. C., 等: DOE E3SM 耦合模型版本 1: 高分辨率的描述和结果, J. Adv. Model. Earth Sy., 11, 4095–4146, [`doi.org/10.1029/2019MS001870`](https://doi.org/10.1029/2019MS001870), 2019. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.25)

Cushman-Roisin, B. 和 Beckers, J.-M.: 地球物理流体动力学导论: 物理和数值方面, Academic press, ISBN 9780080916781, 2011. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.12)

Dalcín, L., Paz, R., 和 Storti, M.: Python 的 MPI, J. Parallel Distr. Com., 65, 1108–1115, 2005. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.32)

Dalcín, L., Paz, R., Storti, M., 和 D’Elía, J.: Python 的 MPI: 性能改进和 MPI-2 扩展, J. Parallel Distr. Com., 68, 655–662, 2008. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.32)

Gevorkyan, M. N., Demidova, A. V., Korolkova, A. V., 和 Kulyabov, D. S.: Julia 科学编程语言的统计显著性能测试, J. Phys Conf. Ser., 1205, 012017, [`doi.org/10.1088/1742-6596/1205/1/012017`](https://doi.org/10.1088/1742-6596/1205/1/012017), 2019. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.2)

Golaz, J.-C., Caldwell, P. M., Van Roekel, L. P., Petersen, M. R., Tang, Q., Wolfe, J. D., Abeshu, G., Anantharaj, V., Asay-Davis, X. S. 和 Bader, D. C.: DOE E3SM 耦合模型版本 1: 标准分辨率的概述和评估, J. Adv. Model. Earth Sy., 11, 2089–2129, [`doi.org/10.1029/2018MS001603`](https://doi.org/10.1029/2018MS001603), 2019. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_altparen.22)

Jiang, J., Lin, P., Wang, J., Liu, H., Chi, X., Hao, H., Wang, Y., Wang, W., and Zhang, L.: 将 LASG/IAP Climate System Ocean Model 移植到 Gpus 使用 OpenAcc, IEEE Access, 7, 154490–154501, [`doi.org/10.1109/ACCESS.2019.2932443`](https://doi.org/10.1109/ACCESS.2019.2932443), 2019. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.5)

Klöckner, A., Pinto, N., Lee, Y., Catanzaro, B., Ivanov, P., 和 Fasih, A.: PyCUDA 和 PyOpenCL: 基于脚本的 GPU 运行时代码生成方法, Parallel Comput., 38, 157–174, [`doi.org/10.1016/j.parco.2011.09.001`](https://doi.org/10.1016/j.parco.2011.09.001), 2012. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.31)

Klöwer, M., Hatfield, S., Croci, M., Düben, P. D., 和 Palmer, T. N.: 使用 16 位加速的流体模拟: 通过将 ShallowWaters.jl 压缩为 Float16，在 A64FX 上接近 4 倍加速, J. Adv. Model. Earth Sy., 14, e2021MS002684, [`doi.org/10.1029/2021MS002684`](https://doi.org/10.1029/2021MS002684), 2022. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.9)

Koldunov, N. V., Aizinger, V., Rakowsky, N., Scholz, P., Sidorenko, D., Danilov, S., 和 Jung, T.: 可扩展性和有限体积海冰-海洋模型的一些优化, 版本 2.0 (FESOM2), 地球科学模型开发, 12, 3991–4012, [`doi.org/10.5194/gmd-12-3991-2019`](https://doi.org/10.5194/gmd-12-3991-2019), 2019. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_altparen.40)

Lam, S. K., Pitrou, A., 和 Seibert, S.: Numba: 一个基于 llvm 的 Python jit 编译器, in: 第二届 LLVM 编译器基础设施在 HPC 中的研讨会, 2015 年 11 月, 1–6, [`doi.org/10.1145/2833157.2833162`](https://doi.org/10.1145/2833157.2833162), 2015. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.30)

Lin, W.-C. 和 McIntosh-Smith, S.: 比较 Julia 和 HPC 的性能可移植并行编程模型, in: 2021 年国际高性能计算机系统性能建模、基准测试和仿真研讨会 (PMBS), IEEE, 美国密苏里州圣路易斯, 2021 年 11 月 15 日, 94–105, [`doi.org/10.1109/PMBS54543.2021.00016`](https://doi.org/10.1109/PMBS54543.2021.00016), 2021. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.2)

NERSC: Perlmutter 架构规范, [`docs.nersc.gov/systems/perlmutter/architecture/`](https://docs.nersc.gov/systems/perlmutter/architecture/) (最后访问：2023 年 6 月 16 日), 2023. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.35)

Norman, M., Lyngaas, I., Bagusetty, A., 和 Berrill, M.: 可移植的 C++ 代码，可以看起来和感觉起来像 Fortran 代码，采用 Yet Another Kernel Launcher (YAKL), 国际并行程序杂志, 51, 209–230, [`doi.org/10.1007/s10766-022-00739-0`](https://doi.org/10.1007/s10766-022-00739-0), 2022. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.7)

Perkel, J. M.: Julia：来因语法而来，为速度而留，自然, 572, 141–142, [`doi.org/10.1038/d41586-019-02310-3`](https://doi.org/10.1038/d41586-019-02310-3), 2019. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.1)

Petersen, M. R., Jacobsen, D. W., Ringler, T. D., Hecht, M. W., 和 Maltrud, M. E.: 在 MPAS-Ocean 模型中评估任意拉格朗日-欧拉垂直坐标方法, 海洋模型., 86, 93–113, [`doi.org/10.1016/j.ocemod.2014.12.004`](https://doi.org/10.1016/j.ocemod.2014.12.004), 2015. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.21)

Petersen, M. R., Asay-Davis, X. S., Berres, A. S., Chen, Q., Feige, N., Hoffman, M. J., Jacobsen, D. W., Jones, P. W., Maltrud, M. E., Price, S. F., Ringler, T. D., Streletz, G. J., Turner, A. K., Van Roekel, L. P., Veneziani, M., Wolfe, J. D., Wolfram, P. J., 和 Woodring, J. L.: 利用 MPAS 和年际 CORE-II 强迫评估 E3SM 的海洋和海冰气候，J. Adv. Model. Earth Sy., 11, 1438–1458, [`doi.org/10.1029/2018MS001373`](https://doi.org/10.1029/2018MS001373), 2019. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_altparen.22)

Petersen, M. R., Bishnu, S., 和 Strauss, R. R.: MPAS-Ocean 浅水性能测试案例，Zenodo [code], [`doi.org/10.5281/zenodo.7439134`](https://doi.org/10.5281/zenodo.7439134), 2022. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.23), [b](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.43), [c](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.44)

Ramadhan, A., Wagner, G. L., Hill, C., Campin, J.-M., Churavy, V., Besard, T., Souza, A., Edelman, A., Ferrari, R., 和 Marshall, J.: Oceananigans.jl：GPU 上快速友好的地球物理流体动力学，J. Open Source Softw., 5, 2018, [`doi.org/10.21105/joss.02018`](https://doi.org/10.21105/joss.02018), 2020. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.8)

Ringler, T. D., Thuburn, J., Klemp, J. B., 和 Skamarock, W. C.: 一个统一的方法来能量守恒和任意结构的 C 网格的位涡动力学，J. Comput. Phys., 229, 3065–3090, 2010. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_text.13), [b](https://gmd.copernicus.org/articles/16/5539/2023/#xref_text.15), [c](https://gmd.copernicus.org/articles/16/5539/2023/#xref_text.17), [d](https://gmd.copernicus.org/articles/16/5539/2023/#xref_text.36)

Ringler, T. D., Petersen, M. R., Higdon, R. L., Jacobsen, D., Jones, P. W., 和 Maltrud, M.: 一个全球海洋建模的多分辨率方法，Ocean Model., 69, 211–232, 2013. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.3), [b](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.10), [c](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.21), [d](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.24)

Shchepetkin, A. F. 和 McWilliams, J. C.: 区域海洋模型系统（ROMS）：一种分裂显式、自由表面、地形跟踪坐标海洋模型，Ocean Model., 9, 347–404, 2005. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_text.27)

Srinath, A.: 使用 nvc++ 和 Cython 加速 GPU 上的 Python，[`developer.nvidia.com/blog/accelerating-python-on-gpus-with-nvc-and-cython/`](https://developer.nvidia.com/blog/accelerating-python-on-gpus-with-nvc-and-cython/)（最后访问：2022 年 12 月 13 日），2022. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.33)

Strauss, R. R.: Julia 中的分层浅水模型, CPU、GPU 和集群硬件上的实现及性能测试, Zenodo [code], [`doi.org/10.5281/zenodo.7493064`](https://doi.org/10.5281/zenodo.7493064), 2023. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.18), [b](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.41), [c](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.47)

Top 500: Top 500 榜单, [`www.top500.org/lists/top500/2022/06`](https://www.top500.org/lists/top500/2022/06)（最后访问日期：2023 年 6 月 16 日）, 2022. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.34)

Thuburn, J., Ringler, T. D., Skamarock, W. C., and Klemp, J. B.: 在任意结构化 C 网格上的地转模式的数值表示, J. Comput. Phys., 228, 8321–8335, 2009. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_text.14), [b](https://gmd.copernicus.org/articles/16/5539/2023/#xref_text.16)

Trott, C. R., Lebrun-Grandié, D., Arndt, D., Ciesko, J., Dang, V., Ellingwood, N., Gayatri, R., Harvey, E., Hollman, D. S. and Ibanez, D.: Kokkos 3: 编程模型扩展适应异构计算的时代, IEEE T. PARALL. Distr., 33, 805–817, [`doi.org/10.1109/TPDS.2021.3097283`](https://doi.org/10.1109/TPDS.2021.3097283), 2022. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.6)

Xu, S., Huang, X., Oey, L.-Y., Xu, F., Fu, H., Zhang, Y., and Yang, G.: POM.gpu-v1.0: 一个基于 GPU 的普林斯顿海洋模型, Geosci. Model Dev., 8, 2815–2827, [`doi.org/10.5194/gmd-8-2815-2015`](https://doi.org/10.5194/gmd-8-2815-2015), 2015. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.4)

Zhao, X.-D., Liang, S.-X., Sun, Z.-C., Zhao, X.-Z., Sun, J.-W., and Liu, Z.-B.: 一个 GPU 加速的有限体积海岸模型, J. Hydrodyn., 29, 679–690, [`doi.org/10.1016/S1001-6058(16)60780-1`](https://doi.org/10.1016/S1001-6058(16)60780-1), 2017. [a](https://gmd.copernicus.org/articles/16/5539/2023/#xref_paren.4)
