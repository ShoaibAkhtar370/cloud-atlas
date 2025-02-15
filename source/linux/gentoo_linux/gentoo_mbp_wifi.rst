.. _gentoo_mbp_wifi:

===================================
Gentoo Linux在MacBook Pro配置Wifi
===================================

Broadcom WiFi
===============

早期MacBook Air 11" 2011版
----------------------------

我曾经使用过MacBook Air 11" 2011版，这款笔记本使用的是Broadcom B43xx系列，是可以使用 ``b43-firmware`` 驱动的，装 ``b43`` 驱动即可:

.. literalinclude:: gentoo_mbp_wifi/macbook_air11_2011_b43
   :language: bash
   :caption: MacBook Air 11" 2011版可以使用 ``b43`` 驱动

Broadcom BCM4360
-------------------

到 :ref:`mbp15_late_2013` 以及我另外一台 MacBook Air 13" ，采用是 Broadcom BCM4360 。这款无线芯片对开源支持不佳，参考 `Linux wireless b43文档 <https://wireless.wiki.kernel.org/en/users/drivers/b43>`_ 可以看到 ``b43`` 驱动不支持BCM4360，建议使用 ``wl`` 驱动。

也就是需要使用闭源的Broadcom驱动( `Apple Macbook Pro Retina - Closed source Broadcom driver <https://wiki.gentoo.org/wiki/Apple_Macbook_Pro_Retina#Closed_source_Broadcom_driver>`_ )

.. literalinclude:: gentoo_mbp_wifi/lspci_k
   :caption: lspci -k 输出硬件信息

内核
========

IEEE 802.11
-------------

至少需要激活 ``cfg80211`` (CONFIG_CFG80211) 和 ``mac80211`` (CONFIG_MAC80211)

.. literalinclude:: gentoo_mbp_wifi/kernel_80211
   :caption: 内核激活 ``cfg80211`` (CONFIG_CFG80211) 和 ``mac80211`` (CONFIG_MAC80211)

.. note::

   由于需要加载firmware，所以 wireless configuration API (CONFIG_CFG80211) 需要配置成模块方式而不是直接buildin

.. note::

   对于私有驱动， **似乎** 需要关闭 ``mac80211`` (CONFIG_MAC80211)

WEXT
-------

``cfg80211 wireless extensions compatibility`` 选项( ``WEXT`` )可以支持传统的 ``wireless-tools`` 和 ``iwconfig`` :

.. literalinclude:: gentoo_mbp_wifi/kernel_wext
   :caption: 内核激活  ``cfg80211 wireless extensions compatibility`` 选项( ``WEXT``  )

.. note::

   我在 Kernel 6.1.12 未找到这个配置项

设备驱动
-----------

注意，建议将驱动编译为内核模块，因为WiFi驱动通常需要firmware，只有作为模块加载时才能使用firmware:

.. literalinclude:: gentoo_mbp_wifi/kernel_wifi_drivers
   :caption: 内核激活相应的驱动内核模块

.. note::

   ``b43`` 可以在内核源码中激活模块方式编译，但是Broadcom4360需要私有驱动 `net-wireless/broadcom-sta <https://packages.gentoo.org/packages/net-wireless/broadcom-sta>`_ 没有编译选项

   对于安装私有驱动，上述设备驱动我全关闭了

   但是参考 `Closed source Broadcom driver <https://wiki.gentoo.org/wiki/Apple_Macbook_Pro_Retina_(early_2013)#Closed_source_Broadcom_driver>`_ 采用激活Intel PRO/Wireless 2100网卡驱动来输出内核的 LIB80211 ::

      Device Drivers
         -> Network device support
            -> Wireless LAN
               -> <*>   Intel PRO/Wireless 2100 Network Connection

LED支持
---------

笔记本内置WiFi没有LED，可忽略

.. literalinclude:: gentoo_mbp_wifi/kernel_wifi_led
   :caption: 内核WiFi的LED支持(数据包收发LED triggers)

Firmware
==========

根据 `gentoo linux wiki: WiFi <https://wiki.gentoo.org/wiki/Wifi>`_ 文档，除了内核模块编译支持之外，WiFi芯片还需要对应firmware:

.. csv-table:: Broadcom无线网卡驱动及firmware
   :file: gentoo_mbp_wifi/broadcom_wifi_driver_firmware.csv
   :widths: 30,20,20,30
   :header-rows: 1

Broadcom BCM4360驱动和Firmware
================================

综上所述，对于 Broadcom BCM4360 实际上就只有安装私有驱动和firmware了，几乎连内核驱动模块都省了:

- 安装 ``wl`` 驱动和firmware:

.. literalinclude:: gentoo_mbp_wifi/broadcom_bcm4360_driver_firmware
   :caption: 安装Broadcom BCM4360的私有驱动和firmware

.. note::

   ``net-wireless/broadcom-sta`` 同时包含了驱动和firmware

``broadcom-sta`` 编译时会检查当前内核编译配置，如果有冲突选项会提示，按提示调整内核编译配置，例如我遇到以下内核配置需要关闭::

   X86_INTEL_LPSS  #位于 "Processor type and features ==> Intel Low Power Subsystem Support"
   PREEMPT_RCU 不能设置为 Preemptible Kernel 的 Preemption Model  #位于General setup ==> RCU Subsystem

``PREEMPT_RCU`` 冲突
=====================

我在最新的 6.1.12 内核安装 ``broadcom-sta`` 遇到报错::

   PREEMPT_RCU: Please do not set the Preemption Model to "Preemptible Kernel"; choose something else. 

这个问题在 `emerge broadcom-sta fails (6.1.12 kernel) due to PREEMPT_RCU <https://forums.gentoo.org/viewtopic-p-8780772.html?sid=5cc5e86da1895dbc2b210c1d15a2d113>`_ 有解决建议:

- ``PREEMPT_RCU`` 不能直接修改(确实，我在配置选项中没有找到)
- 在 ``General setup`` 中 当选择以下 ``preemption models`` 时 ``PREEMPT_RCU`` 会自动选择:

  - Preemptible Kernel (Low-Latency Desktop) # 位于 General setup => Preemption Model
  - Fully Preemptible Kernel (Real-Time) (this might not be available on all CPU architectures)

- 在 ``General setup`` 中如果激活 ``PREEMPT_DYNAMIC`` 就会自动选择

果然，原来我选择激活了 ``Gneeral setup => Preemption behaviour defined on boot`` ，这个选项就是 ``PREEMPT_DYNAMIC=y`` ，就是这个选项导致。真是这个选项激活导致了 ``PREEMPT_RCU`` 出现，我取消这个配置就可以了

参考
=====

- `Apple Macbook Pro Retina - Closed source Broadcom driver <https://wiki.gentoo.org/wiki/Apple_Macbook_Pro_Retina#Closed_source_Broadcom_driver>`_
- `gentoo linux wiki: WiFi <https://wiki.gentoo.org/wiki/Wifi>`_
- `emerge broadcom-sta fails (6.1.12 kernel) due to PREEMPT_RCU <https://forums.gentoo.org/viewtopic-p-8780772.html?sid=5cc5e86da1895dbc2b210c1d15a2d113>`_
