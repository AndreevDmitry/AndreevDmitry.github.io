К счастью для сравнения на https://gitlab.com/sailfishos-porters-ci/mido-ci/-/jobs есть сборка для версии 3.0.2.8. Скачаем ее
```console
HOST:~$ wget https://gitlab.com/sailfishos-porters-ci/mido-ci/-/jobs/187131001/artifacts/download
```
<details>
--2019-07-10 18:55:14--  https://gitlab.com/sailfishos-porters-ci/mido-ci/-/jobs/187131001/artifacts/download<br>
Распознаётся gitlab.com (gitlab.com)… 35.231.145.151<br>
Подключение к gitlab.com (gitlab.com)|35.231.145.151|:443... соединение установлено.<br>
HTTP-запрос отправлен. Ожидание ответа… 302 Found<br>
Адрес: https://storage.googleapis.com/gitlab-gprd-artifacts/2b/37/2b37dab3e5dc662fd745e6804ab3c55b6a5d9cae6ded9f7c165ce28394cf03ba/2019_03_29/187131001/191801863/artifacts.zip?response-content-disposition=attachment%3B%20filename%3D%22artifacts.zip%22%3B%20filename%2A%3DUTF-8%27%27artifacts.zip&response-content-type=application%2Fzip&GoogleAccessId=gitlab-object-storage-prd@gitlab-production.iam.gserviceaccount.com&Signature=e6eAMyGl%2FiOPcJrKeepl7aKAKIj7yyt3B54FLhzVTJt0jcAbgeIzkLFlubuw%0AZscrPzz33It7KMca4vZQG78LxJJ1Y1u76AMBgI6yqJEyiLbR4Pn%2B8Qg7luFL%0A%2FqO9T8Iui5T3M8GKefgi8fLHQBL%2FobO8dsBRLf5tnIAEsP06mjRtH4QZ%2B3Zg%0ADM0dHJpBK6YnwYxQ4GmrHvwScDUZNeZ7KLAw4%2Fi51CLy09oHv8T3BnsfMyBT%0A7M4pw0%2Bd5Y2%2FvaAfxa561w2yKBVKmoulCixvhdEiH8ZX0FHdkTcF3xyaz4Or%0Aw%2BYrOBRXVvhzKg1uuTt7tFwiwQl3xu%2FEmBE4%2FPV0fw%3D%3D&Expires=1562767515 [переход]<br>
--2019-07-10 18:55:15--  https://storage.googleapis.com/gitlab-gprd-artifacts/2b/37/2b37dab3e5dc662fd745e6804ab3c55b6a5d9cae6ded9f7c165ce28394cf03ba/2019_03_29/187131001/191801863/artifacts.zip?response-content-disposition=attachment%3B%20filename%3D%22artifacts.zip%22%3B%20filename%2A%3DUTF-8%27%27artifacts.zip&response-content-type=application%2Fzip&GoogleAccessId=gitlab-object-storage-prd@gitlab-production.iam.gserviceaccount.com&Signature=e6eAMyGl%2FiOPcJrKeepl7aKAKIj7yyt3B54FLhzVTJt0jcAbgeIzkLFlubuw%0AZscrPzz33It7KMca4vZQG78LxJJ1Y1u76AMBgI6yqJEyiLbR4Pn%2B8Qg7luFL%0A%2FqO9T8Iui5T3M8GKefgi8fLHQBL%2FobO8dsBRLf5tnIAEsP06mjRtH4QZ%2B3Zg%0ADM0dHJpBK6YnwYxQ4GmrHvwScDUZNeZ7KLAw4%2Fi51CLy09oHv8T3BnsfMyBT%0A7M4pw0%2Bd5Y2%2FvaAfxa561w2yKBVKmoulCixvhdEiH8ZX0FHdkTcF3xyaz4Or%0Aw%2BYrOBRXVvhzKg1uuTt7tFwiwQl3xu%2FEmBE4%2FPV0fw%3D%3D&Expires=1562767515<br>
Распознаётся storage.googleapis.com (storage.googleapis.com)… 64.233.163.128, 2a00:1450:4010:c06::80<br>
Подключение к storage.googleapis.com (storage.googleapis.com)|64.233.163.128|:443... соединение установлено.<br>
HTTP-запрос отправлен. Ожидание ответа… 200 OK<br>
Длина: 338892013 (323M) [binary/octet-stream]<br>
Сохранение в: «download»<br>
<br>
download                           100%[=============================================================>] 323,19M  17,6MB/s    за 18s<br>
<br>
2019-07-10 18:55:34 (17,7 MB/s) - «download» сохранён [338892013/338892013]<br>
</details><br>

Распакуем скачанный архив, на выходе получим еще один архив
```console
HOST:~$ unzip download -d ~
```
<details>
Archive:  download<br>
  inflating: /home/stalker/sfe-mido-3.0.2.8-devel-20190329/sailfishos-mido-release-3.0.2.8-devel-20190329.zip<br>
</details><br>

Перезагружаемся в twrp и закидываем архив с zip-архив с Sailfish
```console
HOST:~$ adb push sfe-mido-3.0.2.8-devel-20190329/sailfishos-mido-release-3.0.2.8-devel-20190329.zip /sdcard
```
<details>
sfe-mido-3.0.2.8-devel-20190329/sailfishos-mido-release-3.0.2.8-dev...20190329.zip: 1 file pushed. 25.2 MB/s (338794792 bytes in 12.841s)<br>
</details><br>

Подключаемся к телефону по adb и распаковываем zip-архив
```console
HOST:~$ adb shell
adb shell:~ # mkdir -p /sdcard/sailfish_3.0.2.8_gitlab
adb shell:~ # unzip /sdcard/sailfishos-mido-release-3.0.2.8-devel-20190329.zip -d /sdcard/sailfish_3.0.2.8_gitlab/
```
<details>
Archive:  /sdcard/sailfishos-mido-release-3.0.2.8-devel-20190329.zip<br>
  inflating: META-INF/com/google/android/update-binary<br>
  inflating: META-INF/com/google/android/updater-script<br>
  inflating: updater-unpack.sh<br>
  inflating: hybris-boot.img<br>
  inflating: sailfishos-mido-release-3.0.2.8-devel-20190329.tar.bz2<br>
</details><br>

Извлекаем содержимое Sailfish и прошиваем boot
```console
adb shell:~ # /sdcard/busybox_unzip/./busybox tar --numeric-owner -xjf /sdcard/sailfish_3.0.2.8_gitlab/sailfishos-mido-release-3.0.2.8-devel-20190329.tar.bz2  -C /data/.stowaways/sailfishos
adb shell:~ # dd if=/sdcard/sailfish_3.0.2.8_gitlab/hybris-boot.img of=/dev/block/bootdevice/by-name/boot
```
<details>
23320+0 records in<br>
23320+0 records out<br>
11939840 bytes (11.4MB) copied, 0.810947 seconds, 14.0MB/s<br>
</details><br>

Перезагружаемся
```console
adb shell:~ # reboot
```

Графика есть - видим экран выбора языка, снова подлючаемся по telnet
```console
HOST:~$ telnet 192.168.2.15 2323
```
<details>
Trying 192.168.2.15...<br>
Connected to 192.168.2.15.<br>
Escape character is '^]'.<br>
<br>
Welcome to the Mer/SailfishOS Boat loader debug init system.<br>
<br>
Log so far is in /init.log<br>
<br>
To make post-switch_root halt before starting systemd, perform:<br>
  touch /init_enter_debug2<br>
(When run post-switch_root, telnet is on port 2323, not 23)<br>
</details><br>

Соберем необходимую информацию для дальнейшего анализа<br>
Лог ядра рабочей Sailfish 3.0.2.8
```console
sh-3.2# dmesg
```
<details>
[    0.565488] ION heap qsecom created at 0x00000000f5800000 with size 1000000<br>
[    0.566074] msm_bus_fabric_init_driver<br>
[    0.567100] msm_bus_device 580000.ad-hoc-bus: Coresight support absent for bus: 2048<br>
[    0.577277] qcom,qpnp-power-on qpnp-power-on-1: PMIC@SID0 Power-on reason: Triggered from Hard Reset and 'warm' boot<br>
[    0.577318] qcom,qpnp-power-on qpnp-power-on-1: PMIC@SID0: Power-off reason: Triggered from KPDPWR_N (Long Power Key hold)<br>
[    0.577498] input: qpnp_pon as /devices/virtual/input/input0<br>
[    0.578328] pon_spare_reg: no parameters<br>
[    0.578413] qcom,qpnp-power-on qpnp-power-on-14: No PON config. specified<br>
[    0.578469] qcom,qpnp-power-on qpnp-power-on-14: PMIC@SID2 Power-on reason: Triggered from PON1 (secondary PMIC) and 'warm' boot<br>
[    0.578496] qcom,qpnp-power-on qpnp-power-on-14: PMIC@SID2: Power-off reason: Triggered from PS_HOLD (PS_HOLD/MSM controlled shutdown)<br>
[    0.578679] PMIC@SID0: (null) v1.0 options: 2, 2, 0, 0<br>
[    0.578781] PMIC@SID2: PMI8950 v2.0 options: 0, 0, 0, 0<br>
[    0.579402] ipa ipa2_uc_state_check:296 uC interface not initialized<br>
[    0.579415] ipa ipa_sps_irq_control_all:938 EP (2) not allocated.<br>
[    0.579427] ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
[    0.580873] sps:BAM 0x0000000007904000 is registered.<br>
[    0.581359] sps:BAM 0x0000000007904000 (va:0xffffff8001840000) enabled: ver:0x27, number of pipes:20<br>
[    0.584475] IPA driver initialization was successful.<br>
[    0.585439] gdsc_venus: no parameters<br>
[    0.585730] gdsc_mdss: no parameters<br>
[    0.585987] gdsc_jpeg: no parameters<br>
[    0.586408] gdsc_vfe: no parameters<br>
[    0.586852] gdsc_vfe1: no parameters<br>
[    0.587108] gdsc_cpp: no parameters<br>
[    0.587328] gdsc_oxili_gx: no parameters<br>
[    0.587416] gdsc_oxili_gx: supplied by msm_gfx_ldo<br>
[    0.587655] gdsc_venus_core0: fast normal<br>
[    0.587866] gdsc_oxili_cx: no parameters<br>
[    0.588073] gdsc_usb30: no parameters<br>
[    0.588994] mdss_pll_probe: MDSS pll label = MDSS DSI 0 PLL<br>
[    0.589689] dsi_pll_clock_register_8996: Registered DSI PLL ndx=0 clocks successfully<br>
[    0.589723] mdss_pll_probe: MDSS pll label = MDSS DSI 1 PLL<br>
[    0.590847] pll_is_pll_locked_8996: DSI PLL ndx=1 status=0 failed to Lock<br>
[    0.591340] dsi_pll_clock_register_8996: Registered DSI PLL ndx=1 clocks successfully<br>
[    0.591783] msm_iommu 1e00000.qcom,iommu: device apps_iommu (model: 500) mapped at ffffff8001e00000, with 21 ctx banks<br>
[    0.597623] msm_iommu_ctx 1e20000.qcom,iommu-ctx: context adsp_elf using bank 0<br>
[    0.597780] msm_iommu_ctx 1e21000.qcom,iommu-ctx: context adsp_sec_pixel using bank 1<br>
[    0.597935] msm_iommu_ctx 1e22000.qcom,iommu-ctx: context mdp_1 using bank 2<br>
[    0.598089] msm_iommu_ctx 1e23000.qcom,iommu-ctx: context venus_fw using bank 3<br>
[    0.598246] msm_iommu_ctx 1e24000.qcom,iommu-ctx: context venus_sec_non_pixel using bank 4<br>
[    0.598400] msm_iommu_ctx 1e25000.qcom,iommu-ctx: context venus_sec_bitstream using bank 5<br>
[    0.598556] msm_iommu_ctx 1e26000.qcom,iommu-ctx: context venus_sec_pixel using bank 6<br>
[    0.598736] msm_iommu_ctx 1e28000.qcom,iommu-ctx: context pronto_pil using bank 8<br>
[    0.598915] msm_iommu_ctx 1e29000.qcom,iommu-ctx: context q6 using bank 9<br>
[    0.599089] msm_iommu_ctx 1e2a000.qcom,iommu-ctx: context periph_rpm using bank 10<br>
[    0.599269] msm_iommu_ctx 1e2b000.qcom,iommu-ctx: context lpass using bank 11<br>
[    0.599444] msm_iommu_ctx 1e2f000.qcom,iommu-ctx: context adsp_io using bank 15<br>
[    0.599622] msm_iommu_ctx 1e30000.qcom,iommu-ctx: context adsp_opendsp using bank 16<br>
[    0.599799] msm_iommu_ctx 1e31000.qcom,iommu-ctx: context adsp_shared using bank 17<br>
[    0.599978] msm_iommu_ctx 1e32000.qcom,iommu-ctx: context cpp using bank 18<br>
[    0.600170] msm_iommu_ctx 1e33000.qcom,iommu-ctx: context jpeg_enc0 using bank 19<br>
[    0.600354] msm_iommu_ctx 1e34000.qcom,iommu-ctx: context vfe using bank 20<br>
[    0.600529] msm_iommu_ctx 1e35000.qcom,iommu-ctx: context mdp_0 using bank 21<br>
[    0.600706] msm_iommu_ctx 1e36000.qcom,iommu-ctx: context venus_ns using bank 22<br>
[    0.600885] msm_iommu_ctx 1e38000.qcom,iommu-ctx: context ipa using bank 24<br>
[    0.601059] msm_iommu_ctx 1e37000.qcom,iommu-ctx: context access_control using bank 23<br>
[    0.603104] /soc/qcom,cam_smmu/msm_cam_smmu_cb1: could not get #iommu-cells for /soc/qcom,iommu@1e00000<br>
[    0.603146] /soc/qcom,cam_smmu/msm_cam_smmu_cb3: could not get #iommu-cells for /soc/qcom,iommu@1e00000<br>
[    0.603177] /soc/qcom,cam_smmu/msm_cam_smmu_cb4: could not get #iommu-cells for /soc/qcom,iommu@1e00000<br>
[    0.605216] adreno_idler: version 1.1 by arter97<br>
[    0.605302] Advanced Linux Sound Architecture Driver Initialized.<br>
[    0.606477] cfg80211: Calling CRDA to update world regulatory domain<br>
[    0.606502] cfg80211: World regulatory domain updated:<br>
[    0.606513] cfg80211:  DFS Master region: unset<br>
[    0.606521] cfg80211:   (start_freq - end_freq @ bandwidth), (max_antenna_gain, max_eirp), (dfs_cac_time)<br>
[    0.606540] cfg80211:   (2402000 KHz - 2472000 KHz @ 40000 KHz), (N/A, 2000 mBm), (N/A)<br>
[    0.606554] cfg80211:   (2457000 KHz - 2482000 KHz @ 40000 KHz), (N/A, 2000 mBm), (N/A)<br>
[    0.606568] cfg80211:   (2474000 KHz - 2494000 KHz @ 20000 KHz), (N/A, 2000 mBm), (N/A)<br>
[    0.606581] cfg80211:   (5170000 KHz - 5250000 KHz @ 80000 KHz), (N/A, 2000 mBm), (N/A)<br>
[    0.606595] cfg80211:   (5250000 KHz - 5330000 KHz @ 80000 KHz), (N/A, 2000 mBm), (N/A)<br>
[    0.606609] cfg80211:   (5490000 KHz - 5710000 KHz @ 80000 KHz), (N/A, 2000 mBm), (N/A)<br>
[    0.606623] cfg80211:   (5735000 KHz - 5835000 KHz @ 80000 KHz), (N/A, 2000 mBm), (N/A)<br>
[    0.606636] cfg80211:   (57240000 KHz - 63720000 KHz @ 2160000 KHz), (N/A, 0 mBm), (N/A)<br>
[    0.606791] pcie:pcie_init.<br>
[    0.608296] ibb_reg: 4600 -- 6000 mV at 5500 mV<br>
[    0.608573] lab_reg: 4600 -- 6000 mV at 5500 mV<br>
[    0.609166] Switched to clocksource arch_sys_counter<br>
[    0.654200] bcl_peripheral:bcl_perph_init BCL Initialized<br>
[    0.656056] NET: Registered protocol family 2<br>
[    0.656458] TCP established hash table entries: 32768 (order: 6, 262144 bytes)<br>
[    0.656626] TCP bind hash table entries: 32768 (order: 7, 524288 bytes)<br>
[    0.656954] TCP: Hash tables configured (established 32768 bind 32768)<br>
[    0.657004] TCP: reno registered<br>
[    0.657016] UDP hash table entries: 2048 (order: 4, 65536 bytes)<br>
[    0.657072] UDP-Lite hash table entries: 2048 (order: 4, 65536 bytes)<br>
[    0.657255] NET: Registered protocol family 1<br>
[    0.657289] PCI: CLS 0 bytes, default 64<br>
[    0.659484] gcc-mdss-8953 1800000.qcom,gcc-mdss: Registered GCC MDSS clocks.<br>
[    0.659910] Trying to unpack rootfs image as initramfs...<br>
[    0.679682] Freeing initrd memory: 820K<br>
[    0.683872] futex hash table entries: 2048 (order: 5, 131072 bytes)<br>
[    0.683969] Initialise system trusted keyring<br>
[    0.684073] audit: initializing netlink subsys (disabled)<br>
[    0.684122] audit: type=2000 audit(0.683:1): initialized<br>
[    0.684486] vmscan: error setting kswapd cpu affinity mask<br>
[    0.688696] zbud: loaded<br>
[    0.689285] VFS: Disk quotas dquot_6.5.2<br>
[    0.689389] Dquot-cache hash table entries: 512 (order 0, 4096 bytes)<br>
[    0.690309] squashfs: version 4.0 (2009/01/31) Phillip Lougher<br>
[    0.690462] exFAT: Version 1.2.9<br>
[    0.691291] fuse init (API version 7.23)<br>
[    0.691780] msgmni has been set to 5615<br>
[    0.691826] SELinux:  Registering netfilter hooks<br>
[    0.693906] Key type asymmetric registered<br>
[    0.693919] Asymmetric key parser 'x509' registered<br>
[    0.694030] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 247)<br>
[    0.694048] io scheduler noop registered<br>
[    0.694060] io scheduler deadline registered<br>
[    0.694083] io scheduler cfq registered<br>
[    0.694095] io scheduler zen registered<br>
[    0.694113] io scheduler fiops registered<br>
[    0.694125] io scheduler sio registered<br>
[    0.694137] io scheduler maple registered (default)<br>
[    0.694228] io scheduler bfq registered<br>
[    0.694237] BFQ I/O-scheduler: v7r8<br>
[    0.698463] i2c-msm-v2 78b6000.i2c: msm_bus_scale_register_client(mstr-id:86):0x2 (ok)<br>
[    0.698601] i2c-msm-v2 78b6000.i2c: NACK: slave not responding, ensure its powered: msgs(n:2 cur:0 tx) bc(rx:1 tx:1) mode:FIFO slv_addr:0x39 MSTR_STS:0x0d1300c8 OPER:0x00000010<br>
[    0.698650] msm_dba_helper_i2c_read: i2c read failed<br>
[    0.698664] adv7533_read: read err: addr 0x39, reg 0x0, size 0x1<br>
[    0.698674] adv7533_probe: Failed to read chip rev<br>
[    0.698730] adv7533: probe of 2-0039 failed with error -5<br>
[    0.699069] msm_dss_get_res_byname: 'vbif_nrt_phys' resource not found<br>
[    0.699085] mdss_mdp_probe+0x228/0x1108-msm_dss_ioremap_byname: 'vbif_nrt_phys' msm_dss_get_res_byname failed<br>
[    0.699512] mdss_mdp_irq_clk_register: unable to get clk: lut_clk<br>
[    0.700081] No change in context(0==0), skip<br>
[    0.700767] mdss_mdp_pipe_addr_setup: type:0 ftchid:-1 xinid:0 num:0 rect:0 ndx:0x1 prio:0<br>
[    0.700794] mdss_mdp_pipe_addr_setup: type:1 ftchid:-1 xinid:1 num:3 rect:0 ndx:0x8 prio:1<br>
[    0.700808] mdss_mdp_pipe_addr_setup: type:1 ftchid:-1 xinid:5 num:4 rect:0 ndx:0x10 prio:2<br>
[    0.700830] mdss_mdp_pipe_addr_setup: type:2 ftchid:-1 xinid:2 num:6 rect:0 ndx:0x40 prio:3<br>
[    0.700853] mdss_mdp_pipe_addr_setup: type:3 ftchid:-1 xinid:7 num:10 rect:0 ndx:0x400 prio:0<br>
[    0.700872] mdss_mdp_parse_dt_handler: Error from prop qcom,mdss-pipe-sw-reset-off : u32 array read<br>
[    0.700969] mdss_mdp_parse_dt_handler: Error from prop qcom,mdss-ib-factor-overlap : u32 array read<br>
[    0.701246] xlog_status: enable:1, panic:1, dump:2<br>
[    0.701894] mdss_mdp_probe: mdss version = 0x10100000, bootloader display is on, num 1, intf_sel=0x00000100<br>
[    0.703668] mdss_smmu_util_parse_dt_clock: clocks are not defined<br>
[    0.703721] mdss_smmu_probe: iommu v2 domain[0] mapping and clk register successful!<br>
[    0.703755] mdss_smmu_util_parse_dt_clock: clocks are not defined<br>
[    0.703794] mdss_smmu_probe: iommu v2 domain[2] mapping and clk register successful!<br>
[    0.705712] mdss_dsi_ctrl_probe: DSI Ctrl name = MDSS DSI CTRL-0<br>
[    0.706209] mdss_dsi_find_panel_of_node: cmdline:0:qcom,mdss_dsi_nt35532_fhd_video:1:none:cfg:single_dsi panel_name:qcom,mdss_dsi_nt35532_fhd_video<br>
[    0.706277] mdss_dsi_panel_init: Panel Name = nt35532 fhd video mode dsi panel<br>
[    0.706429] mdss_dsi_panel_timing_from_dt: found new timing "qcom,mdss_dsi_nt35532_fhd_video" (ffffffc0ae395000)<br>
[    0.706458] mdss_dsi_parse_dcs_cmds: failed, key=qcom,mdss-dsi-post-panel-on-command<br>
[    0.706474] mdss_dsi_parse_dcs_cmds: failed, key=qcom,mdss-dsi-timing-switch-command<br>
[    0.706488] mdss_dsi_panel_get_dsc_cfg_np: cannot find dsc config node:<br>
[    0.706605] mdss_dsi_parse_panel_features: ulps feature disabled<br>
[    0.706618] mdss_dsi_parse_panel_features: ulps during suspend feature disabled<br>
[    0.706633] mdss_dsi_parse_dms_config: dynamic switch feature enabled: 0<br>
[    0.706660] No valid panel-status-check-mode string<br>
[    0.706768] 1a94000.qcom,mdss_dsi_ctrl0 supply vdd not found, using dummy regulator<br>
[    0.707011] mdss_dsi_parse_ctrl_params:3947 Unable to read qcom,display-id, data=0000000000000000,len=20<br>
[    0.707033] mdss_dsi_parse_gpio_params: bklt_en gpio not specified<br>
[    0.707097] msm_dss_get_res_byname: 'dsi_phy_regulator' resource not found<br>
[    0.707113] mdss_dsi_retrieve_ctrl_resources+0x178/0x1fc-msm_dss_ioremap_byname: 'dsi_phy_regulator' msm_dss_get_res_byname failed<br>
[    0.707130] mdss_dsi_retrieve_ctrl_resources: ctrl_base=ffffff8001808000 ctrl_size=400 phy_base=ffffff800180a400 phy_size=580<br>
[    0.707148] dsi_panel_device_register: Using default BTA for ESD check<br>
[    0.707237] dsi_panel_device_register: Continuous splash enabled<br>
[    0.707504] mdss_register_panel: adding framebuffer device 1a94000.qcom,mdss_dsi_ctrl0<br>
[    0.708817] mdss_dsi_ctrl_probe: Dsi Ctrl-0 initialized, DSI rev:0x10040002, PHY rev:0x2<br>
[    0.708964] mdss_dsi_status_init: DSI status check interval:2500<br>
[    0.709874] mdss_register_panel: adding framebuffer device soc:qcom,mdss_wb_panel<br>
[    0.710413] mdss_fb_probe: fb0: split_mode:0 left:0 right:0<br>
[    0.711081] mdss_fb_register: FrameBuffer[0] 1080x1920 registered successfully!<br>
[    0.711426] mdss_fb_probe: fb1: split_mode:0 left:0 right:0<br>
[    0.711563] mdss_fb_register: FrameBuffer[1] 640x640 registered successfully!<br>
[    0.711644] mdss_mdp_splash_parse_dt: splash mem child node is not present<br>
[    0.711919] mdss_mdp_kcal_store_fb0_ctl panel name nt35532 fhd video mode dsi panel<br>
[    0.711932] mdss_mdp_kcal_store_fb0_ctl panel found...<br>
[    0.711980] kcal_ctrl_init: registered<br>
[    0.714582] IPC_RTR: msm_ipc_router_smd_driver_register Already driver registered IPCRTR<br>
[    0.714621] IPC_RTR: msm_ipc_router_smd_driver_register Already driver registered IPCRTR<br>
[    0.718420] In memshare_probe, Memshare probe success<br>
[    0.720307] msm_rpm_log_probe: OK<br>
[    0.721459] subsys-pil-tz soc:qcom,kgsl-hyp: for a506_zap segments only will be dumped.<br>
[    0.723331] subsys-pil-tz 1de0000.qcom,venus: for venus segments only will be dumped.<br>
[    0.726497] msm_serial_hs module loaded<br>
[    0.737028] platform 1c40000.qcom,kgsl-iommu:gfx3d_secure: assigned reserved memory node secure_region@0<br>
[    0.739693] Boeffla WL blocker: driver version 1.1.0 started<br>
[    0.743717] brd: module loaded<br>
[    0.745802] loop: module loaded<br>
[    0.746131] zram: Added device: zram0<br>
[    0.746581] QSEECOM: qseecom_probe: qseecom.qsee_version = 0x1000000<br>
[    0.746614] QSEECOM: qseecom_retrieve_ce_data: Device does not support PFE<br>
[    0.746629] QSEECOM: qseecom_probe: qseecom clocks handled by other subsystem<br>
[    0.746644] QSEECOM: qseecom_probe: qsee reentrancy support phase is not defined, setting to default 0<br>
[    0.747190] QSEECOM: qseecom_probe: qseecom.whitelist_support = 0<br>
[    0.749374] i2c-core: driver [tabla-i2c-core] using legacy suspend method<br>
[    0.749387] i2c-core: driver [tabla-i2c-core] using legacy resume method<br>
[    0.749470] i2c-core: driver [wcd9xxx-i2c-core] using legacy suspend method<br>
[    0.749482] i2c-core: driver [wcd9xxx-i2c-core] using legacy resume method<br>
[    0.749564] i2c-core: driver [tasha-i2c-core] using legacy suspend method<br>
[    0.749576] i2c-core: driver [tasha-i2c-core] using legacy resume method<br>
[    0.750277] QCE50: __qce_get_device_tree_data: BAM Apps EE is not defined, setting to default 1<br>
[    0.750850] qce 720000.qcedev: Qualcomm Crypto 5.3.3 device found @0x720000<br>
[    0.750866] qce 720000.qcedev: CE device = 0x0<br>
[    0.751084] sps:BAM 0x0000000000704000 is registered.<br>
[    0.751267] sps:BAM 0x0000000000704000 (va:0xffffff8002100000) enabled: ver:0x27, number of pipes:8<br>
[    0.751482] QCE50: qce_sps_init:  Qualcomm MSM CE-BAM at 0x0000000000704000 irq 193<br>
[    0.754278] QCE50: __qce_get_device_tree_data: BAM Apps EE is not defined, setting to default 1<br>
[    0.755118] qcrypto 720000.qcrypto: Qualcomm Crypto 5.3.3 device found @0x720000<br>
[    0.755135] qcrypto 720000.qcrypto: CE device = 0x0<br>
[    0.755425] QCE50: qce_sps_init:  Qualcomm MSM CE-BAM at 0x0000000000704000 irq 193<br>
[    0.757693] qcrypto 720000.qcrypto: qcrypto-ecb-aes<br>
[    0.757783] qcrypto 720000.qcrypto: qcrypto-cbc-aes<br>
[    0.757867] qcrypto 720000.qcrypto: qcrypto-ctr-aes<br>
[    0.757956] qcrypto 720000.qcrypto: qcrypto-ecb-des<br>
[    0.758040] qcrypto 720000.qcrypto: qcrypto-cbc-des<br>
[    0.758124] qcrypto 720000.qcrypto: qcrypto-ecb-3des<br>
[    0.758209] qcrypto 720000.qcrypto: qcrypto-cbc-3des<br>
[    0.758299] qcrypto 720000.qcrypto: qcrypto-xts-aes<br>
[    0.758383] qcrypto 720000.qcrypto: qcrypto-sha1<br>
[    0.758467] qcrypto 720000.qcrypto: qcrypto-sha256<br>
[    0.758554] qcrypto 720000.qcrypto: qcrypto-aead-hmac-sha1-cbc-aes<br>
[    0.758642] qcrypto 720000.qcrypto: qcrypto-aead-hmac-sha1-cbc-des<br>
[    0.758728] qcrypto 720000.qcrypto: qcrypto-aead-hmac-sha1-cbc-3des<br>
[    0.758813] qcrypto 720000.qcrypto: qcrypto-aead-hmac-sha256-cbc-aes<br>
[    0.758899] qcrypto 720000.qcrypto: qcrypto-aead-hmac-sha256-cbc-des<br>
[    0.758984] qcrypto 720000.qcrypto: qcrypto-aead-hmac-sha256-cbc-3des<br>
[    0.759070] qcrypto 720000.qcrypto: qcrypto-hmac-sha1<br>
[    0.759156] qcrypto 720000.qcrypto: qcrypto-hmac-sha256<br>
[    0.759240] qcrypto 720000.qcrypto: qcrypto-aes-ccm<br>
[    0.759328] qcrypto 720000.qcrypto: qcrypto-rfc4309-aes-ccm<br>
[    0.760268] qcom_ice_get_device_tree_data: No vdd-hba-supply regulator, assuming not needed<br>
[    0.760408] ICE IRQ = 194<br>
[    0.761357] SCSI Media Changer driver v0.25<br>
[    0.762417] tun: Universal TUN/TAP device driver, 1.6<br>
[    0.762428] tun: (C) 1999-2004 Max Krasnyansky maxk@qualcomm.com<br>
[    0.762559] PPP generic driver version 2.4.2<br>
[    0.762680] PPP BSD Compression module registered<br>
[    0.762694] PPP Deflate Compression module registered<br>
[    0.762715] PPP MPPE Compression module registered<br>
[    0.762734] NET: Registered protocol family 24<br>
[    0.763340] wcnss_wlan probed in built-in mode<br>
[    0.764084] usbcore: registered new interface driver asix<br>
[    0.764168] usbcore: registered new interface driver ax88179_178a<br>
[    0.764210] usbcore: registered new interface driver cdc_ether<br>
[    0.764252] usbcore: registered new interface driver net1080<br>
[    0.764296] usbcore: registered new interface driver cdc_subset<br>
[    0.764337] usbcore: registered new interface driver zaurus<br>
[    0.764500] usbcore: registered new interface driver cdc_ncm<br>
[    0.764906] msm_sharedmem: sharedmem_register_qmi: qmi init successful<br>
[    0.765702] scm_call failed: func id 0x42000c16, ret: -1, syscall returns: 0x0, 0x0, 0x0<br>
[    0.765717] hyp_assign_table: Failed to assign memory protection, ret = -5<br>
[    0.765731] msm_sharedmem: setup_shared_ram_perms: hyp_assign_phys failed IPA=0x0160x00000000f4900000 size=1572864 err=-5<br>
[    0.765873] msm_sharedmem: msm_sharedmem_probe: Device created for client 'rmtfs'<br>
[    0.771227] msm-dwc3 7000000.ssusb: unable to get dbm device<br>
[    0.772673] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver<br>
[    0.772977] ehci-pci: EHCI PCI platform driver<br>
[    0.773023] ehci-msm: Qualcomm On-Chip EHCI Host Controller<br>
[    0.773354] usbcore: registered new interface driver cdc_acm<br>
[    0.773365] cdc_acm: USB Abstract Control Model driver for USB modems and ISDN adapters<br>
[    0.773423] usbcore: registered new interface driver usb-storage<br>
[    0.773461] usbcore: registered new interface driver ums-alauda<br>
[    0.773499] usbcore: registered new interface driver ums-cypress<br>
[    0.773538] usbcore: registered new interface driver ums-datafab<br>
[    0.773576] usbcore: registered new interface driver ums-freecom<br>
[    0.773613] usbcore: registered new interface driver ums-isd200<br>
[    0.773651] usbcore: registered new interface driver ums-jumpshot<br>
[    0.773690] usbcore: registered new interface driver ums-karma<br>
[    0.773727] usbcore: registered new interface driver ums-sddr09<br>
[    0.773766] usbcore: registered new interface driver ums-sddr55<br>
[    0.773804] usbcore: registered new interface driver ums-usbat<br>
[    0.773893] usbcore: registered new interface driver usbserial<br>
[    0.773939] usbcore: registered new interface driver usb_ehset_test<br>
[    0.774533] gbridge_init: gbridge_init successs.<br>
[    0.774893] mousedev: PS/2 mouse device common for all mice<br>
[    0.775101] usbcore: registered new interface driver xpad<br>
[    0.775115] tony_test:[ft5435_ts_init]<br>
[    0.775545] ~~~~~ ft5435_ts_probe start<br>
[    0.775555] [ft5435_ts_probe]CONFIG_FB is defined<br>
[    0.775564] [ft5435_ts_probe]CONFIG_PM is defined<br>
[    0.775608] [ft5435_get_dt_vkey]000<br>
[    0.775616] [ft5435_get_dt_vkey]111<br>
[    0.775624] [ft5435_get_dt_vkey]222<br>
[    0.775632] [ft5435_get_dt_vkey]333<br>
[    0.775642] [FTS]keycode = 172, x= 500, y=2040<br>
[    0.775652] [FTS]keycode = 139, x= 200, y=2040<br>
[    0.775663] [FTS]keycode = 158, x= 800, y=2040<br>
[    0.775671] [ft5435_get_dt_vkey]5555<br>
[    0.775813] input: ft5435_ts as /devices/soc/78b7000.i2c/i2c-3/3-0038/input/input1<br>
[    0.992251] i2c-msm-v2 78b7000.i2c: msm_bus_scale_register_client(mstr-id:86):0xf (ok)<br>
[    0.992458] ft5435_ts 3-0038: Device ID = 0x54<br>
[    0.992882] sps: BAM device 0x0000000007884000 is not registered yet.<br>
[    0.993092] sps:BAM 0x0000000007884000 is registered.<br>
[    0.993222] sps:BAM 0x0000000007884000 (va:0xffffff8001e60000) enabled: ver:0x19, number of pipes:12<br>
[    0.995141] ft5435_ts 3-0038: report rate = 110Hz<br>
[    0.995728] ft5435_ts 3-0038: Firmware version = 10.0.0<br>
[    0.995881] [Fu]fw_vendor_id=0x51<br>
[    0.995890] upgrade,fts_fw_vendor_id=0x51<br>
[    0.995900] ft5435_fw_upgrade_by_array_data, suspended=0<br>
[    0.995912] ft5435_ts 3-0038: Current firmware: 0x0a.0.0<br>
[    0.995924] ft5435_ts 3-0038: New firmware: 0x0a.0.0<br>
[    0.995934] ft5435_ts 3-0038: Exiting fw upgrade...<br>
[    0.995944] ft5435_fw_upgrade_by_array_data done<br>
[    0.996046] ~~~~~ tp_glove_register enable!!!!!<br>
[    0.996056] [fts]ft5435_fw_LockDownInfo_get_from_boot, fw_vendor_id=0x51<br>
[    1.376274] ft5435_fw_LockDownInfo_get_from_boot, FTS_UPGRADE_LOOP ok is  i = 0<br>
[    1.408292] ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0x34, j=0<br>
[    1.408304] ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0x35, j=1<br>
[    1.408315] ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0x32, j=2<br>
[    1.408326] ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0x01, j=3<br>
[    1.408336] ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0xc6, j=4<br>
[    1.408347] ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0x01, j=5<br>
[    1.408358] ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0x34, j=6<br>
[    1.408369] ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0x01, j=7<br>
[    1.408381] ft5435_fw_LockDownInfo_get_from_boot: reset the tp<br>
[    1.712042] tpd_probe, ft5x46_ctpm_LockDownInfo_get_from_boot, tp_lockdown_info=34353201c6013401<br>
[    1.712077] ft5435_ts 3-0038: Create proc entry success!<br>
[    1.712232] ~~~~~ ft5435_ts_probe end<br>
[    1.712284] [ TSP ] ist30xx_init()<br>
[    1.712565] [ TSP ] ### IMAGIS probe(ver:3.0.0.0, addr:0x50) ###<br>
[    1.712585] [ TSP ] ##### Device tree #####<br>
[    1.712595] [ TSP ]  reset gpio: 64<br>
[    1.712603] [ TSP ]  irq gpio: 65<br>
[    1.712612] [ TSP ] ist30xx_request_gpio<br>
[    1.712621] [ TSP ] unable to request reset gpio: 64<br>
[    1.712631] [ TSP ] Error, ist30xx init driver<br>
[    1.712777] input: s2w_pwrkey as /devices/virtual/input/input2<br>
[    1.713071] [sweep2wake]: sweep2wake_init done<br>
[    1.713198] --------gf_init start.--------<br>
[    1.713790] msm8953-pinctrl 1000000.pinctrl: invalid group "gpio135" for function "blsp_spi6"<br>
[    1.713809] msm8953-pinctrl 1000000.pinctrl: invalid group "gpio136" for function "blsp_spi6"<br>
[    1.713827] msm8953-pinctrl 1000000.pinctrl: invalid group "gpio137" for function "blsp_spi6"<br>
[    1.713845] msm8953-pinctrl 1000000.pinctrl: invalid group "gpio138" for function "blsp_spi6"<br>
[    1.713950] --------gf_probe start.--------<br>
[    1.714170] input: gf3208 as /devices/virtual/input/input3<br>
[    1.714266] --------gf_probe end---OK.--------<br>
[    1.714447]  status = 0x0<br>
[    1.714455] --------gf_init end---OK.--------<br>
[    1.715160] fpc1020 soc:fpc1020: fpc1020_probe: ok<br>
[    1.715294] fpc1020_init OK<br>
[    1.716244] input: hbtp_vm as /devices/virtual/input/input4<br>
[    1.717218] qcom,qpnp-rtc qpnp-rtc-8: rtc core: registered qpnp_rtc as rtc0<br>
[    1.717379] i2c /dev entries driver<br>
[    1.718063] lirc_dev: IR Remote Control driver registered, major 232<br>
[    1.718079] IR NEC protocol handler initialized<br>
[    1.718092] IR RC5(x/sz) protocol handler initialized<br>
[    1.718105] IR RC6 protocol handler initialized<br>
[    1.718118] IR JVC protocol handler initialized<br>
[    1.718130] IR Sony protocol handler initialized<br>
[    1.718143] IR SANYO protocol handler initialized<br>
[    1.718156] IR Sharp protocol handler initialized<br>
[    1.718169] IR LIRC bridge handler initialized<br>
[    1.718182] IR XMP protocol handler initialized<br>
[    1.723538] /soc/qcom,cam_smmu/msm_cam_smmu_cb1: could not get #iommu-cells for /soc/qcom,iommu@1e00000<br>
[    1.723909] /soc/qcom,cam_smmu/msm_cam_smmu_cb3: could not get #iommu-cells for /soc/qcom,iommu@1e00000<br>
[    1.724298] /soc/qcom,cam_smmu/msm_cam_smmu_cb4: could not get #iommu-cells for /soc/qcom,iommu@1e00000<br>
[    1.726909] msm_camera_get_dt_vreg_data:1115 number of entries is 0 or not present in dts<br>
[    1.730413] msm_camera_get_dt_vreg_data:1115 number of entries is 0 or not present in dts<br>
[    1.731188] msm_camera_get_dt_vreg_data:1115 number of entries is 0 or not present in dts<br>
[    1.731886] msm_camera_get_dt_vreg_data:1115 number of entries is 0 or not present in dts<br>
[    1.733946] msm_camera_get_dt_vreg_data:1115 number of entries is 0 or not present in dts<br>
[    1.735185] msm_camera_get_dt_vreg_data:1115 number of entries is 0 or not present in dts<br>
[    1.736429] msm_camera_get_dt_vreg_data:1115 number of entries is 0 or not present in dts<br>
[    1.737084] msm_camera_pinctrl_init:1277 Getting pinctrl handle failed<br>
[    1.737097] msm_actuator_platform_probe:1976 ERR:msm_actuator_platform_probe: Error in reading actuator pinctrl<br>
[    1.737215] qcom,actuator: probe of 1b0c000.qcom,cci:qcom,actuator@0 failed with error -22<br>
[    1.737971] msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
[    1.743457] msm_camera_config_single_vreg : can't find sub reg name<br>
[    1.752451] msm_cci_init:1426: hw_version = 0x10020005<br>
[    1.752485] read_eeprom_memory 158<br>
[    1.752927] msm_cci_irq:1778 MASTER_0 error 0x10000000<br>
[    1.752960] msm_cci_i2c_read:955 read_words = 0, exp words = 1<br>
[    1.752971] msm_cci_i2c_read_bytes:1038 failed rc -22<br>
[    1.752982] read_eeprom_memory 173 error<br>
[    1.752990] msm_eeprom_platform_probe read_eeprom_memory failed<br>
[    1.761220] msm_camera_config_single_vreg : can't find sub reg name<br>
[    1.765437] qcom,eeprom: probe of 1b0c000.qcom,cci:qcom,eeprom@0 failed with error -22<br>
[    1.765889] msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
[    1.772285] msm_camera_config_single_vreg : can't find sub reg name<br>
[    1.781382] msm_cci_init:1426: hw_version = 0x10020005<br>
[    1.781413] read_eeprom_memory 158<br>
[    1.782005] qcom,slave-addr = 0xB0<br>
[    1.782502] qcom,slave-addr = 0xB0<br>
[    1.782999] qcom,slave-addr = 0xB0<br>
[    1.783676] qcom,slave-addr = 0xB0<br>
[    1.784533] qcom,slave-addr = 0xB0<br>
[    1.785030] qcom,slave-addr = 0xB0<br>
[    1.785977] qcom,slave-addr = 0xB0<br>
[    1.786925] qcom,slave-addr = 0xB0<br>
[    2.005044] qcom,slave-addr = 0xB0<br>
[    2.124606] msm_camera_config_single_vreg : can't find sub reg name<br>
[    2.129361] msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
[    2.135727] msm_camera_config_single_vreg : can't find sub reg name<br>
[    2.144821] msm_cci_init:1426: hw_version = 0x10020005<br>
[    2.144852] read_eeprom_memory 158<br>
[    2.145444] qcom,slave-addr = 0xB0<br>
[    2.146031] qcom,slave-addr = 0xB0<br>
[    2.146708] qcom,slave-addr = 0xB0<br>
[    2.147565] qcom,slave-addr = 0xB0<br>
[    2.148603] qcom,slave-addr = 0xB0<br>
[    2.366711] qcom,slave-addr = 0xB0<br>
[    2.477501] qcom,slave-addr = 0xB0<br>
[    2.486397] msm_camera_config_single_vreg : can't find sub reg name<br>
[    2.491206] msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
[    2.512207] msm_cci_init:1426: hw_version = 0x10020005<br>
[    2.512237] read_eeprom_memory 158<br>
[    2.512678] msm_cci_irq:1784 MASTER_1 error 0x40000000<br>
[    2.512711] msm_cci_i2c_read:955 read_words = 0, exp words = 1<br>
[    2.512723] msm_cci_i2c_read_bytes:1038 failed rc -22<br>
[    2.512733] read_eeprom_memory 173 error<br>
[    2.512742] msm_eeprom_platform_probe read_eeprom_memory failed<br>
[    2.532137] qcom,eeprom: probe of 1b0c000.qcom,cci:qcom,eeprom@3 failed with error -22<br>
[    2.532605] msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
[    2.554259] msm_cci_init:1426: hw_version = 0x10020005<br>
[    2.554288] read_eeprom_memory 158<br>
[    2.554729] msm_cci_irq:1784 MASTER_1 error 0x40000000<br>
[    2.554765] msm_cci_i2c_read:955 read_words = 0, exp words = 1<br>
[    2.554776] msm_cci_i2c_read_bytes:1038 failed rc -22<br>
[    2.554786] read_eeprom_memory 173 error<br>
[    2.554795] msm_eeprom_platform_probe read_eeprom_memory failed<br>
[    2.574167] qcom,eeprom: probe of 1b0c000.qcom,cci:qcom,eeprom@4 failed with error -22<br>
[    2.574633] msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
[    2.596285] msm_cci_init:1426: hw_version = 0x10020005<br>
[    2.596315] read_eeprom_memory 158<br>
[    2.736927] msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
[    2.743310] msm_camera_config_single_vreg : can't find sub reg name<br>
[    2.752404] msm_cci_init:1426: hw_version = 0x10020005<br>
[    2.752435] read_eeprom_memory 158<br>
[    2.753026] qcom,slave-addr = 0xB0<br>
[    2.879390] qcom,slave-addr = 0xB0<br>
[    3.005740] qcom,slave-addr = 0xB0<br>
[    3.132086] qcom,slave-addr = 0xB0<br>
[    3.258451] qcom,slave-addr = 0xB0<br>
[    3.384868] qcom,slave-addr = 0xB0<br>
[    3.511257] qcom,slave-addr = 0xB0<br>
[    3.637657] qcom,slave-addr = 0xB0<br>
[    3.772286] msm_camera_config_single_vreg : can't find sub reg name<br>
[    3.777052] msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
[    3.783433] msm_camera_config_single_vreg : can't find sub reg name<br>
[    3.792528] msm_cci_init:1426: hw_version = 0x10020005<br>
[    3.792560] read_eeprom_memory 158<br>
[    3.793001] msm_cci_irq:1778 MASTER_0 error 0x10000000<br>
[    3.793035] msm_cci_i2c_read:955 read_words = 0, exp words = 1<br>
[    3.793046] msm_cci_i2c_read_bytes:1038 failed rc -22<br>
[    3.793057] read_eeprom_memory 173 error<br>
[    3.793066] msm_eeprom_platform_probe read_eeprom_memory failed<br>
[    3.801284] msm_camera_config_single_vreg : can't find sub reg name<br>
[    3.805501] qcom,eeprom: probe of 1b0c000.qcom,cci:qcom,eeprom@7 failed with error -22<br>
[    3.805926] msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
[    3.812296] msm_camera_config_single_vreg : can't find sub reg name<br>
[    3.821381] msm_cci_init:1426: hw_version = 0x10020005<br>
[    3.821411] read_eeprom_memory 158<br>
[    3.822002] qcom,slave-addr = 0xB0<br>
[    3.948404] qcom,slave-addr = 0xB0<br>
[    4.074792] qcom,slave-addr = 0xB0<br>
[    4.201177] qcom,slave-addr = 0xB0<br>
[    4.327576] qcom,slave-addr = 0xB0<br>
[    4.453971] qcom,slave-addr = 0xB0<br>
[    4.580354] qcom,slave-addr = 0xB0<br>
[    4.706737] qcom,slave-addr = 0xB0<br>
[    4.841361] msm_camera_config_single_vreg : can't find sub reg name<br>
[    4.846143] msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
[    4.852528] msm_camera_config_single_vreg : can't find sub reg name<br>
[    4.861627] msm_cci_init:1426: hw_version = 0x10020005<br>
[    4.861659] read_eeprom_memory 158<br>
[    4.862099] msm_cci_irq:1778 MASTER_0 error 0x10000000<br>
[    4.862133] msm_cci_i2c_read:955 read_words = 0, exp words = 1<br>
[    4.862144] msm_cci_i2c_read_bytes:1038 failed rc -22<br>
[    4.862154] read_eeprom_memory 173 error<br>
[    4.862163] msm_eeprom_platform_probe read_eeprom_memory failed<br>
[    4.870381] msm_camera_config_single_vreg : can't find sub reg name<br>
[    4.874601] qcom,eeprom: probe of 1b0c000.qcom,cci:qcom,eeprom@9 failed with error -22<br>
[    4.882316] MSM-CPP cpp_init_hardware:869 CPP HW Version: 0x40030003<br>
[    4.882339] MSM-CPP cpp_init_hardware:887 stream_cnt:0<br>
[    4.894130] __msm_jpeg_init:1537] Jpeg Device id 0<br>
[    4.898537] FG: fg_probe: FG Probe success - FG Revision DIG:3.1 ANA:1.2 PMIC subtype=17<br>
[    4.900163] thermal thermal_zone1: failed to read out thermal zone 1<br>
[    4.900371] thermal thermal_zone2: failed to read out thermal zone 2<br>
[    4.900551] thermal thermal_zone3: failed to read out thermal zone 3<br>
[    4.901052] qpnp_vadc_read: no vadc_chg_vote found<br>
[    4.901064] qpnp_vadc_get_temp: VADC read error with -22<br>
[    4.901076] thermal thermal_zone4: failed to read out thermal zone 4<br>
[    4.922409] device-mapper: uevent: version 1.0.3<br>
[    4.922652] device-mapper: ioctl: 4.28.0-ioctl (2014-09-17) initialised: dm-devel@redhat.com<br>
[    4.922748] device-mapper: req-crypt: dm-req-crypt successfully initalized.<br>
[    4.922807] ------------[ cut here ]------------<br>
[    4.922823] WARNING: CPU: 7 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi/msm8953/fs/sysfs/dir.c:31 sysfs_warn_dup+0x70/0x88()<br>
[    4.922843] sysfs: cannot create duplicate filename '/devices/system/cpu/cpu0/cpufreq/stats'<br>
[    4.922855] Modules linked in:<br>
[    4.922870] CPU: 7 PID: 1 Comm: swapper/0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
[    4.922884] Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
[    4.922897] Call trace:<br>
[    4.922908] [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
[    4.922919] [ffffffc00008a2c4] show_stack+0x20/0x28<br>
[    4.922933] [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
[    4.922944] [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
[    4.922955] [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
[    4.922967] [ffffffc000256774] sysfs_warn_dup+0x70/0x88<br>
[    4.922978] [ffffffc000256f10] internal_create_group+0xc8/0x1f4<br>
[    4.922990] [ffffffc000257068] sysfs_create_group+0x2c/0x38<br>
[    4.923003] [ffffffc0009b3ac0] __cpufreq_stats_create_table.part.3+0x78/0x1ac<br>
[    4.923017] [ffffffc001544404] cpufreq_stats_init+0x180/0x2bc<br>
[    4.923028] [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
[    4.923041] [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
[    4.923053] [ffffffc000e03928] kernel_init+0x20/0xe0<br>
[    4.923063] ---[ end trace baf5d4897624fa09 ]---<br>
[    4.923085] ------------[ cut here ]------------<br>
[    4.923097] WARNING: CPU: 7 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi/msm8953/fs/sysfs/dir.c:31 sysfs_warn_dup+0x70/0x88()<br>
[    4.923116] sysfs: cannot create duplicate filename '/devices/system/cpu/cpu0/cpufreq/stats'<br>
[    4.923129] Modules linked in:<br>
[    4.923141] CPU: 7 PID: 1 Comm: swapper/0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
[    4.923154] Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
[    4.923166] Call trace:<br>
[    4.923176] [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
[    4.923188] [ffffffc00008a2c4] show_stack+0x20/0x28<br>
[    4.923199] [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
[    4.923210] [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
[    4.923221] [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
[    4.923233] [ffffffc000256774] sysfs_warn_dup+0x70/0x88<br>
[    4.923244] [ffffffc000256f10] internal_create_group+0xc8/0x1f4<br>
[    4.923256] [ffffffc000257068] sysfs_create_group+0x2c/0x38<br>
[    4.923268] [ffffffc0009b3ac0] __cpufreq_stats_create_table.part.3+0x78/0x1ac<br>
[    4.923287] [ffffffc001544404] cpufreq_stats_init+0x180/0x2bc<br>
[    4.923299] [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
[    4.923310] [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
[    4.923322] [ffffffc000e03928] kernel_init+0x20/0xe0<br>
[    4.923332] ---[ end trace baf5d4897624fa0a ]---<br>
[    4.923354] ------------[ cut here ]------------<br>
[    4.923367] WARNING: CPU: 7 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi/msm8953/fs/sysfs/dir.c:31 sysfs_warn_dup+0x70/0x88()<br>
[    4.923386] sysfs: cannot create duplicate filename '/devices/system/cpu/cpu0/cpufreq/stats'<br>
[    4.923399] Modules linked in:<br>
[    4.923410] CPU: 7 PID: 1 Comm: swapper/0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
[    4.923424] Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
[    4.923437] Call trace:<br>
[    4.923446] [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
[    4.923458] [ffffffc00008a2c4] show_stack+0x20/0x28<br>
[    4.923469] [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
[    4.923479] [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
[    4.923490] [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
[    4.923502] [ffffffc000256774] sysfs_warn_dup+0x70/0x88<br>
[    4.923513] [ffffffc000256f10] internal_create_group+0xc8/0x1f4<br>
[    4.923525] [ffffffc000257068] sysfs_create_group+0x2c/0x38<br>
[    4.923537] [ffffffc0009b3ac0] __cpufreq_stats_create_table.part.3+0x78/0x1ac<br>
[    4.923550] [ffffffc001544404] cpufreq_stats_init+0x180/0x2bc<br>
[    4.923561] [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
[    4.923573] [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
[    4.923584] [ffffffc000e03928] kernel_init+0x20/0xe0<br>
[    4.923594] ---[ end trace baf5d4897624fa0b ]---<br>
[    4.923615] ------------[ cut here ]------------<br>
[    4.923627] WARNING: CPU: 7 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi/msm8953/fs/sysfs/dir.c:31 sysfs_warn_dup+0x70/0x88()<br>
[    4.923646] sysfs: cannot create duplicate filename '/devices/system/cpu/cpu0/cpufreq/stats'<br>
[    4.923658] Modules linked in:<br>
[    4.923670] CPU: 7 PID: 1 Comm: swapper/0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
[    4.923683] Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
[    4.923695] Call trace:<br>
[    4.923704] [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
[    4.923715] [ffffffc00008a2c4] show_stack+0x20/0x28<br>
[    4.923726] [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
[    4.923737] [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
[    4.923748] [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
[    4.923759] [ffffffc000256774] sysfs_warn_dup+0x70/0x88<br>
[    4.923770] [ffffffc000256f10] internal_create_group+0xc8/0x1f4<br>
[    4.923782] [ffffffc000257068] sysfs_create_group+0x2c/0x38<br>
[    4.923794] [ffffffc0009b3ac0] __cpufreq_stats_create_table.part.3+0x78/0x1ac<br>
[    4.923807] [ffffffc001544404] cpufreq_stats_init+0x180/0x2bc<br>
[    4.923818] [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
[    4.923830] [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
[    4.923841] [ffffffc000e03928] kernel_init+0x20/0xe0<br>
[    4.923851] ---[ end trace baf5d4897624fa0c ]---<br>
[    4.923873] ------------[ cut here ]------------<br>
[    4.923885] WARNING: CPU: 7 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi/msm8953/fs/sysfs/dir.c:31 sysfs_warn_dup+0x70/0x88()<br>
[    4.923904] sysfs: cannot create duplicate filename '/devices/system/cpu/cpu0/cpufreq/stats'<br>
[    4.923916] Modules linked in:<br>
[    4.923928] CPU: 7 PID: 1 Comm: swapper/0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
[    4.923941] Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
[    4.923953] Call trace:<br>
[    4.923963] [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
[    4.923974] [ffffffc00008a2c4] show_stack+0x20/0x28<br>
[    4.923985] [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
[    4.923996] [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
[    4.924007] [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
[    4.924018] [ffffffc000256774] sysfs_warn_dup+0x70/0x88<br>
[    4.924029] [ffffffc000256f10] internal_create_group+0xc8/0x1f4<br>
[    4.924041] [ffffffc000257068] sysfs_create_group+0x2c/0x38<br>
[    4.924053] [ffffffc0009b3ac0] __cpufreq_stats_create_table.part.3+0x78/0x1ac<br>
[    4.924066] [ffffffc001544404] cpufreq_stats_init+0x180/0x2bc<br>
[    4.924077] [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
[    4.924089] [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
[    4.924101] [ffffffc000e03928] kernel_init+0x20/0xe0<br>
[    4.924125] ---[ end trace baf5d4897624fa0d ]---<br>
[    4.924147] ------------[ cut here ]------------<br>
[    4.924159] WARNING: CPU: 7 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi/msm8953/fs/sysfs/dir.c:31 sysfs_warn_dup+0x70/0x88()<br>
[    4.924178] sysfs: cannot create duplicate filename '/devices/system/cpu/cpu0/cpufreq/stats'<br>
[    4.924190] Modules linked in:<br>
[    4.924202] CPU: 7 PID: 1 Comm: swapper/0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
[    4.924215] Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
[    4.924228] Call trace:<br>
[    4.924237] [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
[    4.924248] [ffffffc00008a2c4] show_stack+0x20/0x28<br>
[    4.924259] [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
[    4.924269] [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
[    4.924281] [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
[    4.924292] [ffffffc000256774] sysfs_warn_dup+0x70/0x88<br>
[    4.924303] [ffffffc000256f10] internal_create_group+0xc8/0x1f4<br>
[    4.924315] [ffffffc000257068] sysfs_create_group+0x2c/0x38<br>
[    4.924327] [ffffffc0009b3ac0] __cpufreq_stats_create_table.part.3+0x78/0x1ac<br>
[    4.924340] [ffffffc001544404] cpufreq_stats_init+0x180/0x2bc<br>
[    4.924351] [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
[    4.924363] [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
[    4.924374] [ffffffc000e03928] kernel_init+0x20/0xe0<br>
[    4.924384] ---[ end trace baf5d4897624fa0e ]---<br>
[    4.924404] ------------[ cut here ]------------<br>
[    4.924416] WARNING: CPU: 7 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi/msm8953/fs/sysfs/dir.c:31 sysfs_warn_dup+0x70/0x88()<br>
[    4.924435] sysfs: cannot create duplicate filename '/devices/system/cpu/cpu0/cpufreq/stats'<br>
[    4.924447] Modules linked in:<br>
[    4.924459] CPU: 7 PID: 1 Comm: swapper/0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
[    4.924473] Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
[    4.924485] Call trace:<br>
[    4.924493] [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
[    4.924505] [ffffffc00008a2c4] show_stack+0x20/0x28<br>
[    4.924516] [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
[    4.924526] [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
[    4.924537] [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
[    4.924548] [ffffffc000256774] sysfs_warn_dup+0x70/0x88<br>
[    4.924559] [ffffffc000256f10] internal_create_group+0xc8/0x1f4<br>
[    4.924571] [ffffffc000257068] sysfs_create_group+0x2c/0x38<br>
[    4.924583] [ffffffc0009b3ac0] __cpufreq_stats_create_table.part.3+0x78/0x1ac<br>
[    4.924596] [ffffffc001544404] cpufreq_stats_init+0x180/0x2bc<br>
[    4.924607] [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
[    4.924619] [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
[    4.924630] [ffffffc000e03928] kernel_init+0x20/0xe0<br>
[    4.924640] ---[ end trace baf5d4897624fa0f ]---<br>
[    4.924974] sdhci: Secure Digital Host Controller Interface driver<br>
[    4.924986] sdhci: Copyright(c) Pierre Ossman<br>
[    4.924998] sdhci-pltfm: SDHCI platform and OF driver helper<br>
[    4.925699] qcom_ice_get_pdevice: found ice device ffffffc0ae094200<br>
[    4.925711] qcom_ice_get_pdevice: matching platform device ffffffc0af3ed800<br>
[    4.929675] qcom_ice 7803000.sdcc1ice: QC ICE 2.1.44 device found @0xffffff8001d70000<br>
[    4.930099] sdhci_msm 7824900.sdhci: No vmmc regulator found<br>
[    4.930112] sdhci_msm 7824900.sdhci: No vqmmc regulator found<br>
[    4.930424] mmc0: SDHCI controller on 7824900.sdhci [7824900.sdhci] using 64-bit ADMA in CMDQ mode<br>
[    4.968684] sdhci_msm 7864900.sdhci: sdhci_msm_probe: ICE device is not enabled<br>
[    4.986100] sdhci_msm 7864900.sdhci: No vmmc regulator found<br>
[    4.986122] sdhci_msm 7864900.sdhci: No vqmmc regulator found<br>
[    4.986452] mmc1: SDHCI controller on 7864900.sdhci [7864900.sdhci] using 64-bit ADMA in legacy mode<br>
[    5.008789] mmc0: Out-of-interrupt timeout is 50[ms]<br>
[    5.008800] mmc0: BKOPS_EN equals 0x2<br>
[    5.008809] mmc0: eMMC FW version: 0x07<br>
[    5.008818] mmc0: CMDQ supported: depth: 16<br>
[    5.008827] mmc0: cache barrier support 0 flush policy 0<br>
[    5.018129] cmdq_host_alloc_tdl: desc_size: 768 data_sz: 253952 slot-sz: 24<br>
[    5.018325] mmc0: CMDQ enabled on card<br>
[    5.018339] mmc0: new HS400 MMC card at address 0001<br>
[    5.018632] sdhci_msm_pm_qos_cpu_init (): voted for group #0 (mask=0xf) latency=2<br>
[    5.018649] sdhci_msm_pm_qos_cpu_init (): voted for group #1 (mask=0xf0) latency=2<br>
[    5.018758] mmcblk0: mmc0:0001 RX1BMB 29.1 GiB<br>
[    5.018858] mmcblk0rpmb: mmc0:0001 RX1BMB partition 3 4.00 MiB<br>
[    5.020455]  mmcblk0: p1 p2 p3 p4 p5 p6 p7 p8 p9 p10 p11 p12 p13 p14 p15 p16 p17 p18 p19 p20 p21 p22 p23 p24 p25 p26 p27 p28 p29 p30 p31 p32 p33 p34 p35 p36 p37 p38 p39 p40 p41 p42 p43 p44 p45 p46 p47 p48 p49<br>
[    5.024835] qcom,leds-qpnp: probe of leds-qpnp-21 failed with error -10<br>
[    5.027781] tz_log 8600720.tz-log: Hyp log service is not supported<br>
[    5.028933] hidraw: raw HID events driver (C) Jiri Kosina<br>
[    5.029379] usbcore: registered new interface driver usbhid<br>
[    5.029390] usbhid: USB HID core driver<br>
[    5.029907] ashmem: initialized<br>
[    5.030030] logger: created 256K log 'log_main'<br>
[    5.030146] logger: created 256K log 'log_events'<br>
[    5.030267] logger: created 256K log 'log_radio'<br>
[    5.030382] logger: created 256K log 'log_system'<br>
[    5.030910] qpnp_coincell_charger_show_state: enabled=Y, voltage=3200 mV, resistance=2100 ohm<br>
[    5.035871] bimc-bwmon 408000.qcom,cpu-bwmon: BW HWmon governor registered.<br>
[    5.039383] devfreq soc:qcom,cpubw: Couldn't update frequency transition information.<br>
[    5.039545] devfreq soc:qcom,mincpubw: Couldn't update frequency transition information.<br>
[    5.041666] coresight-fuse a601c.fuse: Fuse initialized<br>
[    5.042160] coresight-cti: probe of 6010000.cti failed with error -1<br>
[    5.042194] coresight-cti: probe of 6011000.cti failed with error -1<br>
[    5.042226] coresight-cti: probe of 6012000.cti failed with error -1<br>
[    5.042258] coresight-cti: probe of 6013000.cti failed with error -1<br>
[    5.042290] coresight-cti: probe of 6014000.cti failed with error -1<br>
[    5.042322] coresight-cti: probe of 6015000.cti failed with error -1<br>
[    5.042354] coresight-cti: probe of 6016000.cti failed with error -1<br>
[    5.042385] coresight-cti: probe of 6017000.cti failed with error -1<br>
[    5.042417] coresight-cti: probe of 6018000.cti failed with error -1<br>
[    5.042450] coresight-cti: probe of 6019000.cti failed with error -1<br>
[    5.042482] coresight-cti: probe of 601a000.cti failed with error -1<br>
[    5.042514] coresight-cti: probe of 601b000.cti failed with error -1<br>
[    5.042546] coresight-cti: probe of 601c000.cti failed with error -1<br>
[    5.042578] coresight-cti: probe of 601d000.cti failed with error -1<br>
[    5.042609] coresight-cti: probe of 601e000.cti failed with error -1<br>
[    5.042641] coresight-cti: probe of 601f000.cti failed with error -1<br>
[    5.042673] coresight-cti: probe of 6198000.cti failed with error -1<br>
[    5.042704] coresight-cti: probe of 6199000.cti failed with error -1<br>
[    5.042736] coresight-cti: probe of 619a000.cti failed with error -1<br>
[    5.042768] coresight-cti: probe of 619b000.cti failed with error -1<br>
[    5.042799] coresight-cti: probe of 61b8000.cti failed with error -1<br>
[    5.042831] coresight-cti: probe of 61b9000.cti failed with error -1<br>
[    5.042864] coresight-cti: probe of 61ba000.cti failed with error -1<br>
[    5.042895] coresight-cti: probe of 61bb000.cti failed with error -1<br>
[    5.042927] coresight-cti: probe of 6128000.cti failed with error -1<br>
[    5.042958] coresight-cti: probe of 6124000.cti failed with error -1<br>
[    5.042990] coresight-cti: probe of 6134000.cti failed with error -1<br>
[    5.043022] coresight-cti: probe of 6139000.cti failed with error -1<br>
[    5.043054] coresight-cti: probe of 613c000.cti failed with error -1<br>
[    5.043086] coresight-cti: probe of 610c000.cti failed with error -1<br>
[    5.043586] coresight-csr 6001000.csr: CSR initialized<br>
[    5.044118] coresight-tmc: probe of 6028000.tmc failed with error -1<br>
[    5.044270] coresight-tmc 6027000.tmc: failed to get flush cti<br>
[    5.044283] coresight-tmc 6027000.tmc: failed to get reset cti<br>
[    5.044498] coresight-tmc 6027000.tmc: TMC initialized<br>
[    5.046243] nidnt boot config: 2<br>
[    5.046252] NIDnT disabled, only sd mode supported.<br>
[    5.046263] coresight-tpiu 6020000.tpiu: NIDnT hw support disabled<br>
[    5.046276] coresight-tpiu 6020000.tpiu: NIDnT on SDCARD only mode<br>
[    5.046350] coresight-tpiu 6020000.tpiu: TPIU initialized<br>
[    5.048087] coresight-replicator 6026000.replicator: REPLICATOR initialized<br>
[    5.048520] coresight-stm: probe of 6002000.stm failed with error -1<br>
[    5.048965] coresight-hwevent 6101000.hwevent: Hardware Event driver initialized<br>
[    5.049913] usbcore: registered new interface driver snd-usb-audio<br>
[    5.063399] msm-pcm-lpa soc:qcom,msm-pcm-lpa: msm_pcm_probe: dev name soc:qcom,msm-pcm-lpa<br>
[    5.068607] u32 classifier<br>
[    5.068620]     Actions configured<br>
[    5.068668] Netfilter messages via NETLINK v0.30.<br>
[    5.068720] nf_conntrack version 0.5.0 (16384 buckets, 65536 max)<br>
[    5.069015] ctnetlink v0.93: registering with nfnetlink.<br>
[    5.069616] xt_time: kernel timezone is -0000<br>
[    5.069670] wireguard: WireGuard 0.0.20180809 loaded. See www.wireguard.com for information.<br>
[    5.069684] wireguard: Copyright (C) 2015-2018 Jason A. Donenfeld Jason@zx2c4.com. All Rights Reserved.<br>
[    5.069874] ip_tables: (C) 2000-2006 Netfilter Core Team<br>
[    5.070028] arp_tables: (C) 2002 David S. Miller<br>
[    5.070080] TCP: cubic registered<br>
[    5.070092] TCP: highspeed registered<br>
[    5.070103] TCP: htcp registered<br>
[    5.070115] TCP: vegas registered<br>
[    5.070126] TCP: veno registered<br>
[    5.070137] TCP: scalable registered<br>
[    5.070149] TCP: lp registered<br>
[    5.070160] TCP: yeah registered<br>
[    5.070171] TCP: illinois registered<br>
[    5.070191] Initializing XFRM netlink socket<br>
[    5.070569] NET: Registered protocol family 10<br>
[    5.071289] mip6: Mobile IPv6<br>
[    5.071317] ip6_tables: (C) 2000-2006 Netfilter Core Team<br>
[    5.071457] sit: IPv6 over IPv4 tunneling driver<br>
[    5.071811] NET: Registered protocol family 17<br>
[    5.071841] NET: Registered protocol family 15<br>
[    5.071882] bridge: automatic filtering via arp/ip/ip6tables has been deprecated. Update your scripts to load br_netfilter if you need this.<br>
[    5.071912] Bridge firewalling registered<br>
[    5.071925] Ebtables v2.0 registered<br>
[    5.072069] l2tp_core: L2TP core driver, V2.0<br>
[    5.072089] l2tp_ppp: PPPoL2TP kernel driver, V2.0<br>
[    5.072103] l2tp_ip: L2TP IP encapsulation support (L2TPv3)<br>
[    5.072129] l2tp_netlink: L2TP netlink interface<br>
[    5.072157] l2tp_eth: L2TP ethernet pseudowire support (L2TPv3)<br>
[    5.072194] l2tp_debugfs: L2TP debugfs support<br>
[    5.072207] l2tp_ip6: L2TP IP encapsulation support for IPv6 (L2TPv3)<br>
[    5.072231] 8021q: 802.1Q VLAN Support v1.8<br>
[    5.072920] NET: Registered protocol family 27<br>
[    5.078137] subsys-pil-tz a21b000.qcom,pronto: for wcnss segments only will be dumped.<br>
[    5.080341] pil-q6v5-mss 4080000.qcom,mss: for modem segments only will be dumped.<br>
[    5.083885] sps:BAM 0x0000000007104000 is registered.<br>
[    5.086888] qpnp-smbcharger qpnp-smbcharger-18: node /soc/qcom,spmi@200f000/qcom,pmi8950@2/qcom,qpnp-smbcharger IO resource absent!<br>
[    5.087066] smbcharger_charger_otg: no parameters<br>
[    5.087174] SMBCHG: smbchg_hvdcp_enable_cb: smbchg_hvdcp_enable_cb  enable 0 last_enable 0<br>
[    5.088277] smbchg_hw_init:read OTG_CFG=78<br>
[    9.588089] SMBCHG: wait_for_usbin_uv: usbin uv didnt go to a lowered state, still at high, tries = 2, rc = 0<br>
[    9.588108] SMBCHG: rerun_apsd: wait for usbin uv failed rc = -22<br>
[    9.644731] ------------[ cut here ]------------<br>
[    9.644749] WARNING: CPU: 0 PID: 79 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi/msm8953/drivers/usb/dwc3/dwc3-msm.c:3590 dwc3_otg_sm_work+0x268/0x610()<br>
[    9.644770] Modules linked in:<br>
[    9.644784] CPU: 0 PID: 79 Comm: kworker/0:1 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
[    9.644798] Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
[    9.644814] Workqueue: events dwc3_otg_sm_work<br>
[    9.644826] Call trace:<br>
[    9.644838] [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
[    9.644849] [ffffffc00008a2c4] show_stack+0x20/0x28<br>
[    9.644862] [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
[    9.644874] [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
[    9.644885] [ffffffc0000a69ac] warn_slowpath_null+0x38/0x44<br>
[    9.644897] [ffffffc000774b58] dwc3_otg_sm_work+0x268/0x610<br>
[    9.644909] [ffffffc0000bf69c] process_one_work+0x25c/0x438<br>
[    9.644920] [ffffffc0000c00c8] worker_thread+0x32c/0x448<br>
[    9.644933] [ffffffc0000c4aa8] kthread+0xf8/0x100<br>
[    9.644942] ---[ end trace baf5d4897624fa10 ]---<br>
[    9.648200] qpnp-smbcharger qpnp-smbcharger-18: node /soc/qcom,spmi@200f000/qcom,pmi8950@2/qcom,qpnp-smbcharger IO resource absent!<br>
[    9.648326] ------------[ cut here ]------------<br>
[    9.648343] WARNING: CPU: 0 PID: 79 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi/msm8953/drivers/usb/dwc3/dwc3-msm.c:3590 dwc3_otg_sm_work+0x268/0x610()<br>
[    9.648380] Modules linked in:<br>
[    9.648395] CPU: 0 PID: 79 Comm: kworker/0:1 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
[    9.648410] Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
[    9.648426] Workqueue: events dwc3_otg_sm_work<br>
[    9.648438] Call trace:<br>
[    9.648449] [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
[    9.648461] [ffffffc00008a2c4] show_stack+0x20/0x28<br>
[    9.648474] [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
[    9.648485] [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
[    9.648497] [ffffffc0000a69ac] warn_slowpath_null+0x38/0x44<br>
[    9.648509] [ffffffc000774b58] dwc3_otg_sm_work+0x268/0x610<br>
[    9.648521] [ffffffc0000bf69c] process_one_work+0x25c/0x438<br>
[    9.648532] [ffffffc0000c00c8] worker_thread+0x32c/0x448<br>
[    9.648545] [ffffffc0000c4aa8] kthread+0xf8/0x100<br>
[    9.648554] ---[ end trace baf5d4897624fa11 ]---<br>
[    9.648790] set_usb_charge_mode_par off = 2<br>
[    9.705089] qpnp-smbcharger qpnp-smbcharger-18: SMBCHG successfully probe Charger version=SCHG_LITE Revision DIG:0.0 ANA:0.1 batt=1 dc=0 usb=0<br>
[    9.706566] EDAC DEVICE0: Giving out device to module soc:arm64-cpu-erp controller cache: DEV soc:arm64-cpu-erp (INTERRUPT)<br>
[    9.707019] ARM64 CPU ERP: Could not find cci-irq IRQ property. Proceeding anyway.<br>
[    9.707032] ARM64 CPU ERP: SBE detection is disabled.<br>
[    9.708258] Registered cp15_barrier emulation handler<br>
[    9.708283] Registered setend emulation handler<br>
[    9.708912] Loading compiled-in X.509 certificates<br>
[    9.710421] Key type encrypted registered<br>
[    9.732165] msm-dwc3 7000000.ssusb: DWC3 in low power mode<br>
[    9.793022] fastrpc soc:qcom,adsprpc-mem: for adsp_rh segments only will be dumped.<br>
[    9.795364] RNDIS_IPA module is loaded.<br>
[    9.795766] file system registered<br>
[    9.795825] mbim_init: initialize 1 instances<br>
[    9.795924] mbim_init: Initialized 1 ports<br>
[    9.797493] rndis_qc_init: initialize rndis QC instance<br>
[    9.797730] Number of LUNs=8<br>
[    9.797742] Mass Storage Function, version: 2009/09/11<br>
[    9.797755] LUN: removable file: (no medium)<br>
[    9.797773] Number of LUNs=1<br>
[    9.797819] LUN: removable file: (no medium)<br>
[    9.797829] Number of LUNs=1<br>
[    9.798144] android_usb gadget: android_usb ready<br>
[    9.799479] input: gpio-keys as /devices/soc/soc:gpio_keys/input/input5<br>
[    9.799864] qcom,qpnp-rtc qpnp-rtc-8: setting system clock to 1970-02-21 20:36:09 UTC (4480569)<br>
[    9.800237] pwm-ir soc:pwm_ir: reg-id = vdd, low-active = 0, use-timer = 0<br>
[    9.800526] Registered IR keymap rc-lirc<br>
[    9.800687] input: pwm-ir as /devices/soc/soc:pwm_ir/rc/rc0/input6<br>
[    9.800807] rc0: pwm-ir as /devices/soc/soc:pwm_ir/rc/rc0<br>
[    9.800993] rc rc0: lirc_dev: driver ir-lirc-codec (pwm-ir) registered at minor = 0<br>
[    9.804436] msm-core initialized without polling period<br>
[    9.807910] parse_cpu_levels: idx 1 276<br>
[    9.807934] calculate_residency: residency  0 for LPM<br>
[    9.808126] parse_cpu_levels: idx 1 286<br>
[    9.808139] calculate_residency: residency  0 for LPM<br>
[    9.811785] qcom,qpnp-flash-led qpnp-flash-led-25: Unable to acquire pinctrl<br>
[    9.813886] rmnet_ipa started initialization<br>
[    9.813899] IPA SSR support = True<br>
[    9.813908] IPA ipa-loaduC = True<br>
[    9.813916] IPA SG support = True<br>
[    9.815889] ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
[    9.815903] ipa ipa2_uc_state_check:301 uC is not loaded<br>
[    9.816897] rmnet_ipa completed initialization<br>
[    9.821745] qcom,cc-debug-8953 1874000.qcom,cc-debug: Registered Debug Mux successfully<br>
[    9.822295] msm8x16_wcd_spmi_probe(6272):slave ID = 0x1<br>
[    9.822328] msm8x16_wcd_spmi_probe(6272):slave ID = 0x1<br>
[    9.833075] msm8952-asoc-wcd c051000.sound: default codec configured<br>
[    9.836607] msm8952-asoc-wcd c051000.sound: ASoC: platform (null) not registered<br>
[    9.836660] msm8952-asoc-wcd c051000.sound: snd_soc_register_card failed (-517)<br>
[    9.837735] apc_mem_acc_corner: disabling<br>
[    9.837748] gfx_mem_acc_corner: disabling<br>
[    9.837791] adv_vreg: disabling<br>
[    9.837800] vdd_vreg: disabling<br>
[    9.837846] clock_late_init: Removing enables held for handed-off clocks<br>
[    9.842760] ALSA device list:<br>
[    9.842769]   No soundcards found.<br>
[    9.843252] Freeing unused kernel memory: 892K<br>
[    9.843292] Freeing alternatives memory: 84K<br>
[   12.254718] EXT3-fs (mmcblk0p49): error: couldn't mount because of unsupported optional features (40)<br>
[   12.255031] EXT2-fs (mmcblk0p49): error: couldn't mount because of unsupported optional features (40)<br>
[   12.257225] EXT4-fs (mmcblk0p49): warning: maximal mount count reached, running e2fsck is recommended<br>
[   12.258252] EXT4-fs (mmcblk0p49): mounted filesystem with ordered data mode. Opts: (null)<br>
[   12.299337] enable_store: android_usb: already disabled<br>
[   12.800962] dwc3 7000000.dwc3: ep0out: Unable to dequeue while in LPM<br>
[   12.801715] rndis_function_bind_config: rndis_function_bind_config MAC: 00:00:00:00:00:00<br>
[   12.801768] android_usb gadget: using random self ethernet address<br>
[   12.801789] android_usb gadget: using random host ethernet address<br>
[   12.802243] rndis0: MAC d2:15:58:dd:84:20<br>
[   12.802254] rndis0: HOST MAC ee:60:50:97:8f:8c<br>
[   12.803344] hid keyboard<br>
[   12.803356] hidg_bind: creating device ffffffc0ac33c600<br>
[   12.803507] hid mouse<br>
[   12.803518] hidg_bind: creating device ffffffc0ac33c800<br>
[   12.833653] IPv6: ADDRCONF(NETDEV_UP): rndis0: link is not ready<br>
[   12.900179] of_batterydata_get_best_profile: qrd_msm8953_sunwoda_atl_4100mah found<br>
[   12.908548] FG: fg_batt_profile_init: Battery SOC: 100, V: 4211553uV<br>
[   12.908995] of_batterydata_get_best_profile: qrd_msm8953_sunwoda_atl_4100mah found<br>
[   13.892690] dwc3 7000000.dwc3: ep0out: Unable to dequeue while in LPM<br>
[   13.892738] hidg_unbind: destroying device ffffffc0ac33c600<br>
[   13.892873] hidg_unbind: destroying device ffffffc0ac33c800<br>
[   13.897664] [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
[   13.916117] [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
[   13.916451] rndis_function_bind_config: rndis_function_bind_config MAC: EE:60:50:97:8F:8C<br>
[   13.916501] android_usb gadget: using random self ethernet address<br>
[   13.916513] android_usb gadget: using previous host ethernet address<br>
[   13.917002] rndis0: MAC 6a:af:46:1c:92:cd<br>
[   13.917013] rndis0: HOST MAC ee:60:50:97:8f:8c<br>
[   13.917111] hid keyboard<br>
[   13.917122] hidg_bind: creating device ffffffc0af5e2e00<br>
[   13.917275] hid mouse<br>
[   13.917286] hidg_bind: creating device ffffffc0af5e2a00<br>
[   14.453542] dwc3 7000000.dwc3: ep0out: Unable to dequeue while in LPM<br>
[   14.453574] hidg_unbind: destroying device ffffffc0af5e2e00<br>
[   14.453718] hidg_unbind: destroying device ffffffc0af5e2a00<br>
[   14.454387] [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
[   14.472139] [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
[   14.472992] rndis_function_bind_config: rndis_function_bind_config MAC: EE:60:50:97:8F:8C<br>
[   14.473040] android_usb gadget: using random self ethernet address<br>
[   14.473053] android_usb gadget: using previous host ethernet address<br>
[   14.473532] rndis0: MAC 0a:91:b7:c6:ca:2d<br>
[   14.473544] rndis0: HOST MAC ee:60:50:97:8f:8c<br>
[   14.473607] hid keyboard<br>
[   14.473620] hidg_bind: creating device ffffffc0ac33cc00<br>
[   14.473774] hid mouse<br>
[   14.473784] hidg_bind: creating device ffffffc0ac33ce00<br>
[   14.511204] random: nonblocking pool is initialized<br>
[   14.636416] dwc3 7000000.dwc3: ep0out: Unable to dequeue while in LPM<br>
[   14.636455] hidg_unbind: destroying device ffffffc0ac33cc00<br>
[   14.636612] hidg_unbind: destroying device ffffffc0ac33ce00<br>
[   14.637229] [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
[   14.652093] [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
[   14.652513] rndis_function_bind_config: rndis_function_bind_config MAC: EE:60:50:97:8F:8C<br>
[   14.652569] android_usb gadget: using random self ethernet address<br>
[   14.652582] android_usb gadget: using previous host ethernet address<br>
[   14.652970] rndis0: MAC 56:82:1f:8c:57:89<br>
[   14.652981] rndis0: HOST MAC ee:60:50:97:8f:8c<br>
[   14.653037] hid keyboard<br>
[   14.653048] hidg_bind: creating device ffffffc0af5e2e00<br>
[   14.653178] hid mouse<br>
[   14.653188] hidg_bind: creating device ffffffc0af5e3200<br>
[   14.694437] dwc3 7000000.dwc3: ep0out: Unable to dequeue while in LPM<br>
[   14.694465] hidg_unbind: destroying device ffffffc0af5e2e00<br>
[   14.694597] hidg_unbind: destroying device ffffffc0af5e3200<br>
[   14.695137] [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
[   14.712084] [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
[   14.713142] rndis_function_bind_config: rndis_function_bind_config MAC: EE:60:50:97:8F:8C<br>
[   14.713193] android_usb gadget: using random self ethernet address<br>
[   14.713206] android_usb gadget: using previous host ethernet address<br>
[   14.713590] rndis0: MAC 86:1a:51:93:c0:33<br>
[   14.713601] rndis0: HOST MAC ee:60:50:97:8f:8c<br>
[   14.713658] hid keyboard<br>
[   14.713669] hidg_bind: creating device ffffffc0ae9b9600<br>
[   14.713798] hid mouse<br>
[   14.713808] hidg_bind: creating device ffffffc0ae9b9800<br>
[   14.751052] IPv6: ADDRCONF(NETDEV_UP): rndis0: link is not ready<br>
[   15.838522] EXT4-fs (mmcblk0p49): re-mounted. Opts: (null)<br>
[   15.852810] preinit: (15.85) Welcome to Sailfish OS 3.0.2.8 (Oulanka)<br>
[   15.867603] preinit: (15.86) is_erase_needed: No : 0<br>
[   15.888807] preinit: (15.88) get_bootstate: USER : 0<br>
[   15.889162] preinit: (15.88) BOOTSTATE = USER<br>
[   15.890007] preinit: (15.88) Booting to default.target<br>
[   15.951547] systemd[1]: systemd 225 running in system mode. (+PAM -AUDIT -SELINUX +IMA -APPARMOR +SMACK -SYSVINIT +UTMP -LIBCRYPTSETUP +GCRYPT -GNUTLS +ACL +XZ -LZ4 -SECCOMP +BLKID -ELFUTILS +KMOD -IDN)<br>
[   15.951835] systemd[1]: Detected architecture arm64.<br>
[   15.952526] systemd[1]: Set hostname to Sailfish.<br>
[   15.952867] systemd[1]: Initializing machine ID from random generator.<br>
[   16.085336] systemd[1]: [/lib/systemd/system/camera-hal.service:8] Executable path is not absolute, ignoring: setprop persist.camera.HAL3.enabled 0<br>
[   16.085544] systemd[1]: camera-hal.service: Service lacks both ExecStart= and ExecStop= setting. Refusing.<br>
[   16.113404] systemd[1]: sys-fs-pstore.mount: Cannot create mount unit for API file system /sys/fs/pstore. Refusing.<br>
[   16.121152] systemd[1]: sys-fs-pstore.mount: Cannot add dependency job, ignoring: Unit sys-fs-pstore.mount failed to load: Invalid argument. See system logs and 'systemctl status sys-fs-pstore.mount' for details.<br>
[   16.122046] systemd[1]: camera-hal.service: Cannot add dependency job, ignoring: Unit camera-hal.service failed to load: Invalid argument. See system logs and 'systemctl status camera-hal.service' for details.<br>
[   16.125517] systemd[1]: Started Dispatch Password Requests to Console Directory Watch.<br>
[   16.125968] systemd[1]: Started Forward Password Requests to Wall Directory Watch.<br>
[   16.126117] systemd[1]: Reached target Swap.<br>
[   16.126255] systemd[1]: Reached target Login Prompts.<br>
[   16.127004] systemd[1]: Created slice Root Slice.<br>
[   16.127233] systemd[1]: Listening on udev Kernel Socket.<br>
[   16.127571] systemd[1]: Listening on Journal Audit Socket.<br>
[   16.127798] systemd[1]: Listening on udev Control Socket.<br>
[   16.128052] systemd[1]: Listening on Journal Socket (/dev/log).<br>
[   16.128275] systemd[1]: Listening on /dev/initctl Compatibility Named Pipe.<br>
[   16.128801] systemd[1]: Created slice User and Session Slice.<br>
[   16.129055] systemd[1]: Listening on Journal Socket.<br>
[   16.129619] systemd[1]: Created slice System Slice.<br>
[   16.129797] systemd[1]: Reached target Slices.<br>
[   16.131191] systemd[1]: Mounting Droid mount for /dev/cpuset...<br>
[   16.132817] systemd[1]: Mounting Debug File System...<br>
[   16.134443] cgroup: new mount options do not match the existing superblock, will be ignored<br>
[   16.135571] systemd[1]: Mounting Temporary Directory...<br>
[   16.137426] systemd[1]: Starting Remount Root and Kernel File Systems...<br>
[   16.139478] systemd[1]: Mounting FFS mount...<br>
[   16.141548] systemd[1]: Starting Create list of required static device nodes for the current kernel...<br>
[   16.143887] systemd[1]: Starting Journal Service...<br>
[   16.146679] systemd[1]: Starting Setup Virtual Console...<br>
[   16.147021] EXT4-fs (mmcblk0p49): re-mounted. Opts: (null)<br>
[   16.150260] systemd[1]: Mounted Debug File System.<br>
[   16.150616] systemd[1]: Mounted FFS mount.<br>
[   16.150800] systemd[1]: Mounted Temporary Directory.<br>
[   16.150975] systemd[1]: Mounted Droid mount for /dev/cpuset.<br>
[   16.153455] systemd[1]: Started Create list of required static device nodes for the current kernel.<br>
[   16.154645] systemd[1]: Started Setup Virtual Console.<br>
[   16.187764] systemd[1]: Started Remount Root and Kernel File Systems.<br>
[   16.257638] systemd[1]: Starting Create System Users...<br>
[   16.259392] systemd[1]: Starting Rebuild Dynamic Linker Cache...<br>
[   16.262483] systemd[1]: Starting Rebuild Hardware Database...<br>
[   16.264448] systemd[1]: Starting Clean RPM db region files at each reboot...<br>
[   16.266371] systemd[1]: Starting Load/Save Random Seed...<br>
[   16.269882] systemd[1]: Started Create System Users.<br>
[   16.280811] systemd[1]: Started Clean RPM db region files at each reboot.<br>
[   16.281916] systemd[1]: Started Load/Save Random Seed.<br>
[   16.316635] systemd[1]: Starting Create Static Device Nodes in /dev...<br>
[   16.334021] systemd[1]: Started Journal Service.<br>
[   16.399673] systemd-journald[686]: Received request to flush runtime journal from PID 1<br>
[   18.221308] EXT4-fs (mmcblk0p12): mounted filesystem with ordered data mode. Opts: (null)<br>
[   18.410373] EXT4-fs (mmcblk0p26): mounted filesystem with ordered data mode. Opts: (null)<br>
[   18.410420] EXT4-fs (mmcblk0p24): mounted filesystem with ordered data mode. Opts: barrier=1<br>
[   18.614209] Loading modules backported from Linux version next-20160324-0-g6f30d29<br>
[   18.614251] Backport generated by backports.git backports-20160324-0-g7dba139<br>
[   18.669043] Bluetooth: Core ver 2.21<br>
[   18.669127] NET: Registered protocol family 31<br>
[   18.669137] Bluetooth: HCI device and connection manager initialized<br>
[   18.669178] Bluetooth: HCI socket layer initialized<br>
[   18.669195] Bluetooth: L2CAP socket layer initialized<br>
[   18.669232] Bluetooth: SCO socket layer initialized<br>
[   18.807329] hcismd_set_enable 0<br>
[   18.807367] Bluetooth: Cannot open the command channel<br>
[   19.014800] droid-hal-init: init first stage started!<br>
[   19.015939] droid-hal-init: init second stage started!<br>
[   19.017638] dwc3 7000000.dwc3: ep0out: Unable to dequeue while in LPM<br>
[   19.017665] hidg_unbind: destroying device ffffffc0ae9b9600<br>
[   19.017828] hidg_unbind: destroying device ffffffc0ae9b9800<br>
[   19.022864] [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
[   19.024135] droid-hal-init: Failed to initialize property area<br>
[   19.024579] droid-hal-init: Running restorecon...<br>
[   19.024671] droid-hal-init: waitpid failed: No child processes<br>
[   19.025322] droid-hal-init: (Loading properties from /default.prop took 0.00s.)<br>
[   19.029303] droid-hal-init: (Parsing /init.environ.rc took 0.00s.)<br>
[   19.031214] droid-hal-init: /init.qcom.rc: 636: invalid keyword 'system'<br>
[   19.031879] droid-hal-init: init.target.rc: 176: invalid keyword 'load_all_props'<br>
[   19.032003] droid-hal-init: (Parsing init.target.rc took 0.00s.)<br>
[   19.032060] droid-hal-init: (Parsing /init.qcom.rc took 0.00s.)<br>
[   19.033060] droid-hal-init: (Parsing /init.usb.configfs.rc took 0.00s.)<br>
[   19.033425] droid-hal-init: (Parsing /init.zygote64_32.rc took 0.00s.)<br>
[   19.034353] droid-hal-init: (Parsing /init.cm.rc took 0.00s.)<br>
[   19.034375] droid-hal-init: (Parsing /init.rc took 0.01s.)<br>
[   19.034886] droid-hal-init: Waiting for /dev/.coldboot_done...<br>
[   19.034916] droid-hal-init: Waiting for /dev/.coldboot_done took 0.00s.<br>
[   19.034956] droid-hal-init: /dev/hw_random not found<br>
[   19.035014] keychord: using input dev qpnp_pon for fevent<br>
[   19.035027] keychord: using input dev ft5435_ts for fevent<br>
[   19.035040] keychord: using input dev s2w_pwrkey for fevent<br>
[   19.035053] keychord: using input dev gf3208 for fevent<br>
[   19.035069] keychord: using input dev gpio-keys for fevent<br>
[   19.035524] droid-hal-init: loglevel: invalid log level'64'<br>
[   19.036472] droid-hal-init: write_file: Unable to open '/proc/sys/kernel/hung_task_timeout_secs': No such file or directory<br>
[   19.037770] droid-hal-init: write_file: Unable to open '/dev/cpuset/foreground/cpus': Permission denied<br>
[   19.037815] droid-hal-init: write_file: Unable to open '/dev/cpuset/foreground/mems': Permission denied<br>
[   19.037945] droid-hal-init: write_file: Unable to open '/dev/cpuset/foreground/boost/cpus': Permission denied<br>
[   19.037988] droid-hal-init: write_file: Unable to open '/dev/cpuset/foreground/boost/mems': Permission denied<br>
[   19.038116] droid-hal-init: write_file: Unable to open '/dev/cpuset/background/cpus': Permission denied<br>
[   19.038157] droid-hal-init: write_file: Unable to open '/dev/cpuset/background/mems': Permission denied<br>
[   19.038288] droid-hal-init: write_file: Unable to open '/dev/cpuset/system-background/cpus': Permission denied<br>
[   19.038330] droid-hal-init: write_file: Unable to open '/dev/cpuset/system-background/mems': Permission denied<br>
[   19.038474] droid-hal-init: write_file: Unable to open '/dev/cpuset/top-app/cpus': Permission denied<br>
[   19.038515] droid-hal-init: write_file: Unable to open '/dev/cpuset/top-app/mems': Permission denied<br>
[   19.039341] Registered swp emulation handler<br>
[   19.040260] droid-hal-init: /dev/hw_random not found<br>
[   19.040366] droid-hal-init: Starting service 'droid_init_done'...<br>
[   19.041286] fs_mgr: Cannot open file fstab.qcom<br>
[   19.041323] droid-hal-init: fs_mgr_mount_all returned an error<br>
[   19.047530] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/atrace.rc took 0.00s.)<br>
[   19.047838] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/bootanim.rc took 0.00s.)<br>
[   19.048459] [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
[   19.048525] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/bootstat.rc took 0.00s.)<br>
[   19.048868] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/debuggerd.rc took 0.00s.)<br>
[   19.051114] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/drmserver.rc took 0.00s.)<br>
[   19.052935] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/dumpstate.rc took 0.00s.)<br>
[   19.054622] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/gatekeeperd.rc took 0.00s.)<br>
[   19.054975] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/init-debug.rc took 0.00s.)<br>
[   19.055448] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/installd.rc took 0.00s.)<br>
[   19.056018] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/logcatd.rc took 0.00s.)<br>
[   19.056626] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/logd.rc took 0.00s.)<br>
[   19.057041] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/mediacodec.rc took 0.00s.)<br>
[   19.057435] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/mediadrmserver.rc took 0.00s.)<br>
[   19.057986] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/mediaextractor.rc took 0.00s.)<br>
[   19.059071] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/mtpd.rc took 0.00s.)<br>
[   19.059470] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/perfprofd.rc took 0.00s.)<br>
[   19.059854] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/racoon.rc took 0.00s.)<br>
[   19.060236] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/rild.rc took 0.00s.)<br>
[   19.060716] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/servicemanager.rc took 0.00s.)<br>
[   19.060774] droid-hal-init: Could not import file '/usr/libexec/droid-hybris/system/etc/init/superuser.rc'<br>
[   19.061278] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/uncrypt.rc took 0.00s.)<br>
[   19.061662] droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/vdc.rc took 0.00s.)<br>
[   19.064472] droid-hal-init: fs_mgr_mount_all returned unexpected error 255<br>
[   19.064900] droid-hal-init: Starting service 'logd'...<br>
[   19.065959] droid-hal-init: couldn't write 2616 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.070156] droid-hal-init: (Loading properties from /system/build.prop took 0.00s.)<br>
[   19.070242] droid-hal-init: (Loading properties from /vendor/build.prop took 0.00s.)<br>
[   19.070288] droid-hal-init: (Loading properties from /factory/factory.prop took 0.00s.)<br>
[   19.070534] fs_mgr: Cannot open file /fstab.qcom<br>
[   19.070558] droid-hal-init: unable to read fstab /fstab.qcom: No such file or directory<br>
[   19.070886] droid-hal-init: Starting service 'debuggerd'...<br>
[   19.071375] droid-hal-init: do_start: Service debuggerd64 not found<br>
[   19.071403] droid-hal-init: do_start: Service vold not found<br>
[   19.071438] droid-hal-init: couldn't write 2618 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.074364] droid-hal-init: Not bootcharting.<br>
[   19.113702] wcnss_wlan triggered by userspace<br>
[   19.114275] wcnss_pm_qos_add_request: add request<br>
[   19.114304] wcnss_pm_qos_update_request: update request 100<br>
[   19.114761] wcnss_notif_cb: wcnss notification event: 2<br>
[   19.131995] subsys-pil-tz a21b000.qcom,pronto: wcnss: loading from 0x000000008e700000 to 0x000000008ed42000<br>
[   19.132124] wcnss_notif_cb: wcnss notification event: 6<br>
[   19.138005] wcnss: IRIS Reg: 04000004<br>
[   19.140640] audit: type=1305 audit(1552425864.019:2): audit_pid=2616 old=0 auid=4294967295 ses=4294967295 subj=kernel res=1<br>
[   19.140748] logd.auditd: start<br>
[   19.140790] logd.klogd: 19140765095<br>
[   19.193094] droid-hal-init: Starting service 'exec 1 (/system/bin/tzdatacheck)'...<br>
[   19.204299] capability: warning: `ofonod' uses 32-bit capabilities (legacy support in use)<br>
[   19.204460] droid-hal-init: Service 'exec 1 (/system/bin/tzdatacheck)' (pid 2663) exited with status 0<br>
[   19.211196] type=1305 audit(1552425864.019:3): audit_rate_limit=20 old=0 auid=4294967295 ses=4294967295 subj=kernel res=1<br>
[   19.211755] type=1701 audit(1552425864.055:4): auid=4294967295 uid=0 gid=0 ses=4294967295 subj=kernel pid=2658 comm="vdc" exe="/system/bin/vdc" sig=6<br>
[   19.228105] droid-hal-init: Starting service 'perfd'...<br>
[   19.229112] droid-hal-init: couldn't write 2670 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.236574] droid-hal-init: write_file: Unable to open '/sys/block/dm-0/queue/read_ahead_kb': No such file or directory<br>
[   19.236632] droid-hal-init: write_file: Unable to open '/sys/block/dm-1/queue/read_ahead_kb': No such file or directory<br>
[   19.241261] droid-hal-init: Starting service 'sysinit'...<br>
[   19.242181] droid-hal-init: (Loading properties from /data/local.prop took 0.00s.)<br>
[   19.354108] droid-hal-init: Starting service 'logd-reinit'...<br>
[   19.354735] droid-hal-init: couldn't write 2708 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.361462] subsys-pil-tz c200000.qcom,lpass: adsp: loading from 0x000000008d600000 to 0x000000008e700000<br>
[   19.365914] logd.daemon: reinit<br>
[   19.398368] subsys-pil-tz a21b000.qcom,pronto: wcnss: Brought out of reset<br>
[   19.639905] subsys-pil-tz c200000.qcom,lpass: adsp: Brought out of reset<br>
[   19.676961] type=1325 audit(1552425864.555:5): table=filter family=2 entries=4<br>
[   19.677267] type=1300 audit(1552425864.555:5): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40 a3=2e5868 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
[   19.677777] type=1327 audit(1552425864.555:5): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
[   19.678056] type=1320 audit(1552425864.555:5):<br>
[   19.682328] type=1325 audit(1552425864.559:6): table=mangle family=2 entries=6<br>
[   19.682613] type=1300 audit(1552425864.559:6): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40 a3=2e5b18 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
[   19.682824] type=1327 audit(1552425864.559:6): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
[   19.683049] type=1320 audit(1552425864.559:6):<br>
[   19.685683] subsys-pil-tz c200000.qcom,lpass: Subsystem error monitoring/handling services are up<br>
[   19.685746] subsys-pil-tz c200000.qcom,lpass: adsp: Power/Clock ready interrupt received<br>
[   19.685833] L-Notify: Generel: 7<br>
[   19.686126] droid-hal-init: Service 'sysinit' (pid 2671) exited with status 0<br>
[   19.686331] droid-hal-init: Service 'logd-reinit' (pid 2708) exited with status 0<br>
[   19.686463] sensors-ssc soc:qcom,msm-ssc-sensors: slpi_loader_do: pil get failed,<br>
[   19.686480] sensors-ssc soc:qcom,msm-ssc-sensors: slpi_loader_do: SLPI image loading failed<br>
[   19.686943] diag: In diag_send_feature_mask_update, control channel is not open, p: 1, 0000000000000000<br>
[   19.687738] msm8x16_wcd_spmi_probe(6272):slave ID = 0x1<br>
[   19.689459] droid-hal-init: Starting service 'irsc_util'...<br>
[   19.690122] droid-hal-init: Starting service 'rmt_storage'...<br>
[   19.690740] droid-hal-init: Starting service 'tftp_server'...<br>
[   19.690818] type=1325 audit(1552425864.567:7): table=nat family=2 entries=5<br>
[   19.691275] msm8x16_wcd_spmi_probe(6272):slave ID = 0x1<br>
[   19.691426] droid-hal-init: Starting service 'config_bt_addr'...<br>
[   19.692222] droid-hal-init: Starting service 'config_bluetooth'...<br>
[   19.692979] droid-hal-init: Starting service 'sensors'...<br>
[   19.693705] droid-hal-init: Starting service 'gx_fpd'...<br>
[   19.694472] droid-hal-init: Starting service 'per_mgr'...<br>
[   19.695217] droid-hal-init: Starting service 'servicemanager'...<br>
[   19.695593] droid-hal-init: couldn't write 2841 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.696469] droid-hal-init: Starting service 'cnd'...<br>
[   19.697097] droid-hal-init: Starting service 'netmgrd'...<br>
[   19.697747] droid-hal-init: couldn't write 2835 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.698242] droid-hal-init: Starting service 'qti'...<br>
[   19.701952] droid-hal-init: couldn't write 2834 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.702042] droid-hal-init: Starting service 'ril-daemon2'...<br>
[   19.702077] droid-hal-init: couldn't write 2842 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.702162] droid-hal-init: couldn't write 2840 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.702723] droid-hal-init: Starting service 'thermal-engine'...<br>
[   19.703414] droid-hal-init: Starting service 'wcnss-service'...<br>
[   19.704186] droid-hal-init: Starting service 'imsqmidaemon'...<br>
[   19.704958] droid-hal-init: couldn't write 2845 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.704967] droid-hal-init: Starting service 'adsprpcd'...<br>
[   19.705698] droid-hal-init: Starting service 'hvdcp_opti'...<br>
[   19.706043] type=1300 audit(1552425864.567:7): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40 a3=2e59c0 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
[   19.706216] type=1327 audit(1552425864.567:7): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
[   19.706413] droid-hal-init: Starting service 'energy-awareness'...<br>
[   19.706420] type=1320 audit(1552425864.567:7):<br>
[   19.706614] droid-hal-init: couldn't write 2855 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.707126] droid-hal-init: Starting service 'drm'...<br>
[   19.707878] droid-hal-init: Starting service 'installd'...<br>
[   19.707950] droid-hal-init: couldn't write 2859 to /dev/cpuset/foreground/tasks: No space left on device<br>
[   19.708830] type=1325 audit(1552425864.587:8): table=filter family=10 entries=4<br>
[   19.709082] droid-hal-init: couldn't write 2847 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.709742] droid-hal-init: couldn't write 2846 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.710157] droid-hal-init: couldn't write 2848 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.717379] msm8952-asoc-wcd c051000.sound: default codec configured<br>
[   19.717477] droid-hal-init: couldn't write 2853 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.721094] type=1300 audit(1552425864.587:8): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=29 a2=40 a3=2e5a28 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
[   19.725079] droid-hal-init: Starting service 'mediacodec'...<br>
[   19.725765] droid-hal-init: Starting service 'mediadrm'...<br>
[   19.726418] droid-hal-init: Starting service 'mediaextractor'...<br>
[   19.726577] msm8952-asoc-wcd c051000.sound: ASoC: platform (null) not registered<br>
[   19.726632] msm8952-asoc-wcd c051000.sound: snd_soc_register_card failed (-517)<br>
[   19.727116] droid-hal-init: Starting service 'ril-daemon'...<br>
[   19.737525] type=1327 audit(1552425864.587:8): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
[   19.737806] type=1320 audit(1552425864.587:8):<br>
[   19.744234] msm8952-asoc-wcd c051000.sound: default codec configured<br>
[   19.751750] wcd-spmi-core msm8x16_wcd_codec-11: Error: regulator not found:cdc-vdd-spkdrv<br>
[   19.754937] droid-hal-init: couldn't write 2867 to /dev/cpuset/foreground/tasks: No space left on device<br>
[   19.755488] droid-hal-init: couldn't write 2866 to /dev/cpuset/foreground/tasks: No space left on device<br>
[   19.756094] droid-hal-init: couldn't write 2865 to /dev/cpuset/foreground/tasks: No space left on device<br>
[   19.757681] droid-hal-init: Starting service 'minimedia'...<br>
[   19.758353] droid-hal-init: Starting service 'minisf'...<br>
[   19.759083] droid-hal-init: Starting service 'miniaf'...<br>
[   19.769566] type=1325 audit(1552425864.643:9): table=mangle family=10 entries=6<br>
[   19.769806] type=1300 audit(1552425864.643:9): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=29 a2=40 a3=2e5dc0 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
[   19.769933] type=1327 audit(1552425864.643:9): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
[   19.770116] type=1320 audit(1552425864.643:9):<br>
[   19.789058] droid-hal-init: Service 'irsc_util' (pid 2832) exited with status 0<br>
[   19.789327] droid-hal-init: Service 'config_bt_addr' (pid 2836) exited with status 0<br>
[   19.798280] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet0/accept_ra': No such file or directory<br>
[   19.798300] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no dapm match for MM_DL5 -- MultiMedia5 -- TERT_MI2S_RX Audio Mixer<br>
[   19.798304] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route MM_DL5 - MultiMedia5 - TERT_MI2S_RX Audio Mixer<br>
[   19.798337] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no dapm match for MM_DL8 -- MultiMedia8 -- TERT_MI2S_RX Audio Mixer<br>
[   19.798340] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route MM_DL8 - MultiMedia8 - TERT_MI2S_RX Audio Mixer<br>
[   19.798420] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet1/accept_ra': No such file or directory<br>
[   19.798476] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet2/accept_ra': No such file or directory<br>
[   19.798529] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet3/accept_ra': No such file or directory<br>
[   19.798583] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet4/accept_ra': No such file or directory<br>
[   19.798639] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet5/accept_ra': No such file or directory<br>
[   19.798691] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet6/accept_ra': No such file or directory<br>
[   19.798721] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no dapm match for INT_FM_TX -- INTERNAL_FM_TX -- SEC_MI2S_RX Port Mixer<br>
[   19.798743] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route INT_FM_TX - INTERNAL_FM_TX - SEC_MI2S_RX Port Mixer<br>
[   19.798744] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet7/accept_ra': No such file or directory<br>
[   19.798775] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio0/accept_ra': No such file or directory<br>
[   19.798827] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio1/accept_ra': No such file or directory<br>
[   19.798881] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio2/accept_ra': No such file or directory<br>
[   19.798936] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio3/accept_ra': No such file or directory<br>
[   19.798991] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio4/accept_ra': No such file or directory<br>
[   19.799043] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio5/accept_ra': No such file or directory<br>
[   19.799096] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio6/accept_ra': No such file or directory<br>
[   19.799149] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio7/accept_ra': No such file or directory<br>
[   19.799203] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_usb0/accept_ra': No such file or directory<br>
[   19.799256] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_usb1/accept_ra': No such file or directory<br>
[   19.799307] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_usb2/accept_ra': No such file or directory<br>
[   19.799361] droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_usb3/accept_ra': No such file or directory<br>
[   19.801782] droid-hal-init: sanitize_path: Failed to locate path /dev/block/bootdevice/by-name/rawdump: No such file or directory<br>
[   19.801835] droid-hal-init: do_chown: sanitize_path failed for path: /dev/block/bootdevice/by-name/rawdump prefix:/dev/block/<br>
[   19.801889] droid-hal-init: sanitize_path: Failed to locate path /dev/block/bootdevice/by-name/rawdump: No such file or directory<br>
[   19.801916] droid-hal-init: do_chmod: failed for /dev/block/bootdevice/by-name/rawdump..prefix(/dev/block/) match err<br>
[   19.802020] droid-hal-init: write_file: Unable to open '/dev/cpuset/top-app/cpus': Permission denied<br>
[   19.802069] droid-hal-init: write_file: Unable to open '/dev/cpuset/top-app/boost/cpus': No such file or directory<br>
[   19.802119] droid-hal-init: write_file: Unable to open '/dev/cpuset/foreground/cpus': Permission denied<br>
[   19.802171] droid-hal-init: write_file: Unable to open '/dev/cpuset/foreground/boost/cpus': Permission denied<br>
[   19.804823] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no dapm match for VOICEMMODE1_DL -- VoiceMMode1 -- SEC_MI2S_RX_Voice Mixer<br>
[   19.804866] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route VOICEMMODE1_DL - VoiceMMode1 - SEC_MI2S_RX_Voice Mixer<br>
[   19.804906] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no dapm match for VOICEMMODE2_DL -- VoiceMMode2 -- SEC_MI2S_RX_Voice Mixer<br>
[   19.804925] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route VOICEMMODE2_DL - VoiceMMode2 - SEC_MI2S_RX_Voice Mixer<br>
[   19.807536] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no source widget found for AUDIO_REF_EC_UL3 MUX<br>
[   19.807574] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route AUDIO_REF_EC_UL3 MUX - direct - MM_UL3<br>
[   19.812173] type=1006 audit(1552425864.675:10): pid=2844 uid=0 subj=kernel old-auid=4294967295 auid=100000 old-ses=4294967295 ses=1 res=1<br>
[   19.816476] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no dapm match for SEC_MI2S_TX -- SEC_MI2S_TX -- SLIMBUS_0_RX Port Mixer<br>
[   19.816520] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route SEC_MI2S_TX - SEC_MI2S_TX - SLIMBUS_0_RX Port Mixer<br>
[   19.816523] droid-hal-init: write_file: Unable to open '/dev/cpuset/background/cpus': Permission denied<br>
[   19.816585] droid-hal-init: write_file: Unable to open '/dev/cpuset/system-background/cpus': Permission denied<br>
[   19.817221] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no dapm match for VOICE2_STUB_DL -- Voice2 Stub -- INTERNAL_BT_SCO_RX_Voice Mixer<br>
[   19.817262] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route VOICE2_STUB_DL - Voice2 Stub - INTERNAL_BT_SCO_RX_Voice Mixer<br>
[   19.817455] droid-hal-init: write_file: Unable to open '/dev/cpuset/camera-daemon/cpus': Permission denied<br>
[   19.817523] droid-hal-init: write_file: Unable to open '/dev/cpuset/camera-daemon/mems': Permission denied<br>
[   19.818563] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no sink widget found for SENARY_MI2S_TX<br>
[   19.818596] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route BE_IN - direct - SENARY_MI2S_TX<br>
[   19.820848] droid-hal-init: Starting service 'dpmd'...<br>
[   19.821663] droid-hal-init: Starting service 'loc_launcher'...<br>
[   19.827758] droid-hal-init: Starting service 'qcom-sh'...<br>
[   19.828931] droid-hal-init: Starting service 'qseeproxydaemon'...<br>
[   19.829329] sysmon-qmi: sysmon_clnt_svc_arrive: Connection established between QMI handle and adsp's SSCTL service<br>
[   19.829856] droid-hal-init: Starting service 'qcamerasvr'...<br>
[   19.831428] droid-hal-init: couldn't write 2931 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.841382] droid-hal-init: couldn't write 2924 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.841774] droid-hal-init: Starting service 'audiod'...<br>
[   19.842577] droid-hal-init: Starting service 'gatekeeperd'...<br>
[   19.843687] droid-hal-init: couldn't write 2925 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.848270] droid-hal-init: couldn't write 2930 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.850932] droid-hal-init: Starting service 'perfprofd'...<br>
[   19.851148] type=1325 audit(1552425864.727:11): table=filter family=10 entries=4<br>
[   19.852074] droid-hal-init: Starting service 'console'...<br>
[   19.852877] droid-hal-init: Starting service 'imsdatadaemon'...<br>
[   19.853757] droid-hal-init: Starting service 'per_proxy'...<br>
[   19.857160] droid-hal-init: couldn't write 2946 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.870057] droid-hal-init: couldn't write 2953 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.870071] droid-hal-init: couldn't write 2954 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.870484] droid-hal-init: couldn't write 2951 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   19.874702] type=1300 audit(1552425864.727:11): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=29 a2=40 a3=2e6dd0 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
[   19.874843] type=1327 audit(1552425864.727:11): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
[   19.875000] type=1320 audit(1552425864.727:11):<br>
[   19.886700] subsys-pil-tz a21b000.qcom,pronto: wcnss: Power/Clock ready interrupt received<br>
[   19.887434] wcnss_notif_cb: wcnss notification event: 7<br>
[   19.896698] type=1325 audit(1552425864.767:12): table=filter family=10 entries=8<br>
[   19.896963] type=1300 audit(1552425864.767:12): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=29 a2=40 a3=2e7308 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
[   19.897052] type=1327 audit(1552425864.767:12): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
[   19.897217] type=1320 audit(1552425864.767:12):<br>
[   19.900722] subsys-pil-tz a21b000.qcom,pronto: Subsystem error monitoring/handling services are up<br>
[   19.902625] wcnss_notif_cb: wcnss notification event: 3<br>
[   19.902647] wcnss_pm_qos_update_request: update request -1<br>
[   19.902658] wcnss_pm_qos_remove_request: remove request<br>
[   19.905026] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to create SEC_MI2S_RX Port Mixer debugfs file<br>
[   19.906410] hcismd_set_enable 1<br>
[   19.906451] Bluetooth: Cannot open the command channel<br>
[   19.906849] apr_tal:Q6 Is Up<br>
[   19.916092] 'opened /dev/adsprpc-smd c 227 0'<br>
[   19.919164] type=1325 audit(1552425864.795:13): table=filter family=2 entries=4<br>
[   19.920766] diag: In diag_send_feature_mask_update, control channel is not open, p: 2, 0000000000000000<br>
[   19.935688] type=1300 audit(1552425864.795:13): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40 a3=2e6b00 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
[   19.935816] type=1327 audit(1552425864.795:13): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
[   19.935980] type=1320 audit(1552425864.795:13):<br>
[   19.955255] type=1325 audit(1552425864.831:14): table=filter family=2 entries=8<br>
[   19.955551] type=1300 audit(1552425864.831:14): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40 a3=2e6f60 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
[   19.955645] type=1327 audit(1552425864.831:14): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
[   19.955808] type=1320 audit(1552425864.831:14):<br>
[   19.957920] msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: mux SLIM_0_RX AANC MUX has no paths<br>
[   19.975693] wcd-spmi-core msm8x16_wcd_codec-11: ASoC: mux RX3 MIX1 INP3 has no paths<br>
[   19.976385] wcd-spmi-core msm8x16_wcd_codec-11: ASoC: mux RX2 MIX1 INP3 has no paths<br>
[   19.979558] msm_thermal:set_enabled enabled = 0<br>
[   19.982541] audit: audit_lost=1 audit_rate_limit=20 audit_backlog_limit=64<br>
[   19.982579] audit: rate limit exceeded<br>
[   19.984273] type=1325 audit(1552425864.859:15): table=filter family=10 entries=9<br>
[   20.020771] subsys-restart: __subsystem_get(): Changing subsys fw_name to modem<br>
[   20.021723] IPA received MPSS BEFORE_POWERUP<br>
[   20.022157] ipa ipa2_uc_state_check:301 uC is not loaded<br>
[   20.022198] ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
[   20.023575] IPA BEFORE_POWERUP handling is complete<br>
[   20.049009] msm8952-asoc-wcd c051000.sound: control 3:0:0:IEC958 Playback PCM Stream:0 is already present<br>
[   20.049053] msm-hdmi-dba-codec-rx soc:qcom,msm-hdmi-dba-codec-rx: ASoC: Failed to add IEC958 Playback PCM Stream: -16<br>
[   20.049072] msm_dba_get_probed_device: Device not found (adv7533, 0)<br>
[   20.049084] msm_dba_register_client: Device not found (adv7533, 0)<br>
[   20.049096] msm_hdmi_dba_codec_rx_init_dba: error in registering audio client 0<br>
[   20.066646] wcnss_wlan_ctrl_probe: SMD ctrl channel up<br>
[   20.067010] wcnss: version 01050102<br>
[   20.067025] wcnss: schedule dnld work for pronto<br>
[   20.067215] wcnss: build version CNSS-PR-4-0-00328<br>
[   20.110063] droid-hal-init: Starting service 'msm_irqbalance'...<br>
[   20.144825] wlan: module is from the staging directory, the quality is unknown, you have been warned.<br>
[   20.145001] pil-q6v5-mss 4080000.qcom,mss: modem: loading from 0x0000000086c00000 to 0x000000008c200000<br>
[   20.172368] input: msm8953-snd-card-mtp Headset Jack as /devices/soc/c051000.sound/sound/card0/input7<br>
[   20.172719] input: msm8953-snd-card-mtp Button Jack as /devices/soc/c051000.sound/sound/card0/input8<br>
[   20.296482] wlan: loading driver v3.0.11.66<br>
[   20.331275] droid-hal-init: Service 'droid_init_done' (pid 2611) exited with status 0<br>
[   20.337305] droid-hal-init: Service 'config_bluetooth' (pid 2837) exited with status 0<br>
[   20.453164] droid-hal-init: Service 'energy-awareness' (pid 2858) exited with status 0<br>
[   20.572616] pil-q6v5-mss 4080000.qcom,mss: Debug policy not present - msadp. Continue.<br>
[   20.577941] pil-q6v5-mss 4080000.qcom,mss: Loading MBA and DP (if present) from 0x00000000f4b00000 to 0x00000000f4c00000 size 100000<br>
[   20.715486] Bluetooth: BNEP (Ethernet Emulation) ver 1.3<br>
[   20.715498] Bluetooth: BNEP filters: protocol multicast<br>
[   20.715531] Bluetooth: BNEP socket layer initialized<br>
[   20.717183] pil-q6v5-mss 4080000.qcom,mss: MBA boot done<br>
[   20.921329] hcismd_set_enable 1<br>
[   20.921369] Bluetooth: Cannot open the command channel<br>
[   20.967885] QSEECOM: qseecom_load_app: App (goodixfp) does'nt exist, loading apps for first time<br>
[   21.080292] QSEECOM: qseecom_load_app: App with id 5 (goodixfp) now loaded<br>
[   21.082329] --------driver_init_partial start.--------<br>
[   21.082358] --------gf_parse_dts start.--------<br>
[   21.082407] goodix_fp soc:goodix_fp: goodix,gpio_reset 140<br>
[   21.082426] goodix_fp soc:goodix_fp: goodix,gpio_irq 48<br>
[   21.082705] msm8953-pinctrl 1000000.pinctrl: invalid group "gpio135" for function "blsp_spi6"<br>
[   21.082726] msm8953-pinctrl 1000000.pinctrl: invalid group "gpio136" for function "blsp_spi6"<br>
[   21.082745] msm8953-pinctrl 1000000.pinctrl: invalid group "gpio137" for function "blsp_spi6"<br>
[   21.082764] msm8953-pinctrl 1000000.pinctrl: invalid group "gpio138" for function "blsp_spi6"<br>
[   21.082861] found pin control goodixfp_reset_reset<br>
[   21.082865] found pin control goodixfp_reset_active<br>
[   21.082868] found pin control goodixfp_irq_active<br>
[   21.082886] goodix_fp soc:goodix_fp: Selected 'goodixfp_reset_active'<br>
[   21.082906] goodix_fp soc:goodix_fp: Selected 'goodixfp_irq_active'<br>
[   21.082917] --------gf_parse_dts end---OK.--------<br>
[   21.083536] ------------[ cut here ]------------<br>
[   21.083564] WARNING: CPU: 7 PID: 2841 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi/msm8953/kernel/irq/manage.c:448 __enable_irq+0x4c/0x90()<br>
[   21.083584] Unbalanced enable for IRQ 55<br>
[   21.083592] Modules linked in: bnep(O) wlan(C+) hci_smd(O) bluetooth(O) compat(O) skcipher(O) autofs4<br>
[   21.083613] CPU: 7 PID: 2841 Comm: gx_fpd Tainted: G        WC O   3.18.105-ElectraBlue-11.0-mido #3<br>
[   21.083617] Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
[   21.083621] Call trace:<br>
[   21.083633] [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
[   21.083638] [ffffffc00008a2c4] show_stack+0x20/0x28<br>
[   21.083645] [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
[   21.083651] [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
[   21.083655] [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
[   21.083660] [ffffffc000102a18] __enable_irq+0x4c/0x90<br>
[   21.083665] [ffffffc000102ae0] enable_irq+0x84/0xac<br>
[   21.083673] [ffffffc00082eaa4] gf_ioctl+0x268/0x4e8<br>
[   21.083679] [ffffffc0001f2368] do_vfs_ioctl+0x4ec/0x5e0<br>
[   21.083683] [ffffffc0001f24c8] SyS_ioctl+0x6c/0x94<br>
[   21.083687] ---[ end trace baf5d4897624fa13 ]---<br>
[   21.083715] goodix_fp soc:goodix_fp: Selected 'goodixfp_reset_reset'<br>
[   21.085227] type=1325 audit(1552425865.959:47): table=mangle family=2 entries=14<br>
[   21.085493] type=1300 audit(1552425865.959:47): arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=0 a2=40 a3=7fa4814800 items=0 ppid=2847 pid=3581 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
[   21.085708] type=1327 audit(1552425865.959:47): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D74006D616E676C65002D4E0071636F6D5F716F735F72657365745F504F5354524F5554494E47<br>
[   21.085860] type=1320 audit(1552425865.959:47):<br>
[   21.086746] goodix_fp soc:goodix_fp: Selected 'goodixfp_reset_active'<br>
[   21.086764] goodix_fp soc:goodix_fp: IRQ after reset 0<br>
[   21.095782] type=1325 audit(1552425865.971:48): table=mangle family=2 entries=16<br>
[   21.096353] type=1300 audit(1552425865.971:48): arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=0 a2=40 a3=7fa6e15400 items=0 ppid=2847 pid=3590 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
[   21.096740] type=1327 audit(1552425865.971:48): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D74006D616E676C65002D4100504F5354524F5554494E47002D6A0071636F6D5F716F735F72657365745F504F5354524F5554494E47<br>
[   21.096998] type=1320 audit(1552425865.971:48):<br>
[   21.112628] type=1325 audit(1552425865.987:49): table=mangle family=10 entries=6<br>
[   21.113952] type=1300 audit(1552425865.987:49): arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=29 a2=40 a3=7f8fe3f000 items=0 ppid=2847 pid=3598 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="ip6tables" exe="/system/bin/ip6tables" subj=kernel key=(null)<br>
[   21.114426] type=1327 audit(1552425865.987:49): proctitle=2F73797374656D2F62696E2F6970367461626C6573002D77002D74006D616E676C65002D4E0071636F6D5F716F735F72657365745F504F5354524F5554494E47<br>
[   21.114638] type=1320 audit(1552425865.987:49):<br>
[   21.129781] type=1325 audit(1552425866.007:50): table=mangle family=10 entries=8<br>
[   21.130082] type=1300 audit(1552425866.007:50): arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=29 a2=40 a3=7f88847000 items=0 ppid=2847 pid=3602 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="ip6tables" exe="/system/bin/ip6tables" subj=kernel key=(null)<br>
[   21.130419] type=1327 audit(1552425866.007:50): proctitle=2F73797374656D2F62696E2F6970367461626C6573002D77002D74006D616E676C65002D4100504F5354524F5554494E47002D6A0071636F6D5F716F735F72657365745F504F5354524F5554494E47<br>
[   21.130581] type=1320 audit(1552425866.007:50):<br>
[   21.145180] type=1325 audit(1552425866.019:51): table=mangle family=2 entries=17<br>
[   21.145428] type=1300 audit(1552425866.019:51): arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=0 a2=40 a3=7f9943b000 items=0 ppid=2847 pid=3609 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
[   21.145733] type=1327 audit(1552425866.019:51): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D74006D616E676C65002D4E0071636F6D5F716F735F66696C7465725F504F5354524F5554494E47<br>
[   21.145933] type=1320 audit(1552425866.019:51):<br>
[   21.155280] audit: audit_lost=136 audit_rate_limit=20 audit_backlog_limit=64<br>
[   21.155320] audit: rate limit exceeded<br>
[   21.237408] msm_pm_qos_add_request: add request<br>
[   21.300258] msm_cci_init:1426: hw_version = 0x10020005<br>
[   21.300897] s5k5e8_ofilm probe succeeded<br>
[   21.326378] msm_cci_init:1426: hw_version = 0x10020005<br>
[   21.326612] s5k3l8_ofilm probe succeeded<br>
[   21.344633] msm_csid_init: CSID_VERSION = 0x30050001<br>
[   21.345299] msm_csid_irq CSID2_IRQ_STATUS_ADDR = 0x800<br>
[   21.345760] msm_csid_init: CSID_VERSION = 0x30050001<br>
[   21.347046] msm_csid_irq CSID0_IRQ_STATUS_ADDR = 0x800<br>
[   21.377279] pil-q6v5-mss 4080000.qcom,mss: modem: Brought out of reset<br>
[   21.448334] Doesn't support control clock.<br>
[   21.448345] This kernel doesn't support control clk in AP<br>
[   21.452637] droid-hal-init: Service 'qcom-sh' (pid 2927) exited with status 0<br>
[   21.459108] pil-q6v5-mss 4080000.qcom,mss: modem: Power/Clock ready interrupt received<br>
[   21.459111] pil-q6v5-mss 4080000.qcom,mss: Subsystem error monitoring/handling services are up<br>
[   21.462142] IPA received MPSS AFTER_POWERUP<br>
[   21.462151] IPA AFTER_POWERUP handling is complete<br>
[   21.463559] M-Notify: General: 7<br>
[   21.468534] IRQ has been disabled.<br>
[   21.468583] goodix_fp soc:goodix_fp: Selected 'goodixfp_reset_reset'<br>
[   21.471611] goodix_fp soc:goodix_fp: Selected 'goodixfp_reset_active'<br>
[   21.471632] goodix_fp soc:goodix_fp: IRQ after reset 0<br>
[   21.483060] QSEECOM: qseecom_unload_app: App id 5 now unloaded<br>
[   21.486830] gf:[info]  entergf_cleanup<br>
[   21.486865] gf:remove irq_gpio success<br>
[   21.486876] gf:remove reset_gpio success<br>
[   21.486925] gf:gx  fingerprint_pinctrl  release success<br>
[   21.487028] ---- power off ----<br>
[   21.487588] droid-hal-init: Service 'gx_fpd' is being killed...<br>
[   21.487737] droid-hal-init: Starting service 'fingerprintd'...<br>
[   21.488827] droid-hal-init: couldn't write 3701 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   21.489520] droid-hal-init: Service 'gx_fpd' (pid 2841) killed by signal 9<br>
[   21.489544] droid-hal-init: Service 'gx_fpd' (pid 2841) killing any children in process group<br>
[   21.534816] [info] goodix_fb_state_chg_callback go to the goodix_fb_state_chg_callback value = 16<br>
[   21.562471] sysmon-qmi: sysmon_clnt_svc_arrive: Connection established between QMI handle and modem's SSCTL service<br>
[   21.565919] subsys-pil-tz soc:qcom,kgsl-hyp: a506_zap: loading from 0x000000008f000000 to 0x000000008f002000<br>
[   21.574466] MSM-CPP cpp_init_hardware:869 CPP HW Version: 0x40030003<br>
[   21.574480] MSM-CPP cpp_init_hardware:887 stream_cnt:0<br>
[   21.583204] subsys-pil-tz soc:qcom,kgsl-hyp: a506_zap: Brought out of reset<br>
[   21.585526] devfreq soc:qcom,kgsl-busmon: Couldn't update frequency transition information.<br>
[   21.666123] apr_tal:Modem Is Up<br>
[   21.678111] diag: In diag_send_feature_mask_update, control channel is not open, p: 0, 0000000000000000<br>
[   21.713863] Sending QMI_IPA_INIT_MODEM_DRIVER_REQ_V01<br>
[   21.714032] ipa-wan handle_indication_req:133 not send indication<br>
[   21.716577] ipa ipa_uc_response_hdlr:461 IPA uC loaded<br>
[   21.716646] ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
[   21.717031] QMI_IPA_INIT_MODEM_DRIVER_REQ_V01 response received<br>
[   21.719387] ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
[   21.721216] ipa ipa_uc_wdi_event_log_info_handler:322 WDI feature missing 0x1<br>
[   21.721241] ipa ipa_uc_ntn_event_log_info_handler:39 NTN feature missing 0x1<br>
[   21.721288] ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
[   21.776413] msm_pm_qos_update_request: update request 100<br>
[   21.786274] msm_csid_init: CSID_VERSION = 0x30050001<br>
[   21.787048] msm_csid_irq CSID0_IRQ_STATUS_ADDR = 0x800<br>
[   21.791019] MSM-CPP cpp_init_hardware:869 CPP HW Version: 0x40030003<br>
[   21.791053] MSM-CPP cpp_init_hardware:887 stream_cnt:0<br>
[   21.807708] msm_cci_init:1426: hw_version = 0x10020005<br>
[   21.815465] msm_pm_qos_update_request: update request -1<br>
[   21.928415] hcismd_set_enable 1<br>
[   21.928456] Bluetooth: Cannot open the command channel<br>
[   22.125821] scm_call failed: func id 0x42000c16, ret: -1, syscall returns: 0x0, 0x0, 0x0<br>
[   22.125860] hyp_assign_table: Failed to assign memory protection, ret = -5<br>
[   22.125865] memshare: hyp_assign_phys failed size=2097152 err=-5<br>
[   22.245900] droid-hal-init: Starting service 'ims_rtp_daemon'...<br>
[   22.246475] droid-hal-init: Starting service 'imscmservice'...<br>
[   22.246939] droid-hal-init: couldn't write 3996 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   22.246960] droid-hal-init: couldn't write 3997 to /dev/cpuset/system-background/tasks: No space left on device<br>
[   22.584051] wcnss: NV download<br>
[   22.590996] wcnss: NV bin size: 31719, total_fragments: 11<br>
[   22.591058] wcnss: no space available for smd frame<br>
[   22.612339] wcnss: no space available for smd frame<br>
[   22.636110] wcnss: no space available for smd frame<br>
[   22.660124] wcnss: no space available for smd frame<br>
[   22.933714] hcismd_set_enable 1<br>
[   22.933878] Bluetooth: opening HCI-SMD channel :APPS_RIVA_BT_ACL<br>
[   22.933914] Bluetooth: opening HCI-SMD channel :APPS_RIVA_BT_CMD<br>
[   22.933999] Bluetooth: HCI device registration is starting<br>
[   22.948936] msm_pm_qos_update_request: update request 100<br>
[   22.962244] msm_csid_init: CSID_VERSION = 0x30050001<br>
[   22.963000] msm_csid_irq CSID2_IRQ_STATUS_ADDR = 0x800<br>
[   22.965383] MSM-CPP cpp_init_hardware:869 CPP HW Version: 0x40030003<br>
[   22.965396] MSM-CPP cpp_init_hardware:887 stream_cnt:0<br>
[   22.984952] msm_pm_qos_update_request: update request -1<br>
[   23.020883] msm_cci_init:1426: hw_version = 0x10020005<br>
[   23.067164] ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
[   23.068818] ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
[   23.071072] ipa-wan ipa_wwan_ioctl:1542 get AGG size 8192 count 10<br>
[   23.071641] ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
[   23.073516] ipa ipa_assign_policy:3112 get close-by 8192<br>
[   23.073543] ipa ipa_assign_policy:3118 set rx_buff_sz 7808<br>
[   23.073553] ipa ipa_assign_policy:3141 set aggr_limit 6<br>
[   23.084644] ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data0) register to IPA<br>
[   23.113688] type=1325 audit(1552425867.991:55): table=mangle family=2 entries=20<br>
[   23.113941] type=1300 audit(1552425867.991:55): arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40 a3=7f8d839e00 items=0 ppid=2847 pid=4206 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
[   23.116667] type=1327 audit(1552425867.991:55): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D74006D616E676C65002D4100505245524F5554494E47002D6D00736F636B6574002D2D6E6F77696C6463617264002D2D726573746F72652D736B6D61726B002D6A00414343455054<br>
[   23.116889] type=1320 audit(1552425867.991:55):<br>
[   23.118713] droid-hal-init: property_set("ro.ril.svlte1x", "false") failed<br>
[   23.118875] droid-hal-init: property_set("ro.ril.svdo", "false") failed<br>
[   23.125763] type=1325 audit(1552425868.003:56): table=mangle family=2 entries=21<br>
[   23.126069] type=1300 audit(1552425868.003:56): arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40 a3=7fb5c39e00 items=0 ppid=2847 pid=4207 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
[   23.126342] type=1327 audit(1552425868.003:56): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D74006D616E676C65002D4400505245524F5554494E47002D6D00736F636B6574002D2D6E6F77696C6463617264002D2D726573746F72652D736B6D61726B002D6A00414343455054<br>
[   23.127660] type=1320 audit(1552425868.003:56):<br>
[   23.135375] IPC_RTR: msm_ipc_router_bind: slim_daemon Do not have permissions<br>
[   23.138279] ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data1) register to IPA<br>
[   23.139102] type=1325 audit(1552425868.015:57): table=raw family=2 entries=3<br>
[   23.139492] type=1300 audit(1552425868.015:57): arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40 a3=7fa7640000 items=0 ppid=2847 pid=4212 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
[   23.142738] type=1327 audit(1552425868.015:57): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D7400726177002D4E006E6D5F6D646D707278795F7261775F707265<br>
[   23.143665] type=1320 audit(1552425868.015:57):<br>
[   23.151166] type=1325 audit(1552425868.027:58): table=mangle family=2 entries=20<br>
[   23.151846] type=1300 audit(1552425868.027:58): arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40 a3=7f7961d000 items=0 ppid=2847 pid=4226 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
[   23.152721] type=1327 audit(1552425868.027:58): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D74006D616E676C65002D4E006E6D5F6D646D707278795F6D6E676C5F706F7374<br>
[   23.152924] type=1320 audit(1552425868.027:58):<br>
[   23.162360] type=1325 audit(1552425868.039:59): table=mangle family=2 entries=22<br>
[   23.162684] type=1300 audit(1552425868.039:59): arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40 a3=7f8301d000 items=0 ppid=2847 pid=4228 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
[   23.163041] type=1327 audit(1552425868.039:59): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D74006D616E676C65002D4E006E6D5F6D646D707278795F6D6E676C5F7072655F737069<br>
[   23.163242] type=1320 audit(1552425868.039:59):<br>
[   23.174790] audit: audit_lost=148 audit_rate_limit=20 audit_backlog_limit=64<br>
[   23.174825] audit: rate limit exceeded<br>
[   23.214348] ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data2) register to IPA<br>
[   23.278922] ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data3) register to IPA<br>
[   23.343194] ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data4) register to IPA<br>
[   23.410484] ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data5) register to IPA<br>
[   23.465517] ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data6) register to IPA<br>
[   23.534539] ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data7) register to IPA<br>
[   23.938552] hcismd_set_enable 1<br>
[   23.942670] hcismd_set_enable 1<br>
[   23.942705] Bluetooth: HCI device un-registration going on<br>
[   23.944262] Bluetooth: Frame for unknown HCI device (hdev=NULL)<br>
[   23.944293] Bluetooth: Frame for unknown HCI device (hdev=NULL)<br>
[   24.086383] hcismd_set_enable 0<br>
[   24.086495] Bluetooth: opening HCI-SMD channel :APPS_RIVA_BT_ACL<br>
[   24.086526] Bluetooth: opening HCI-SMD channel :APPS_RIVA_BT_CMD<br>
[   24.086605] Bluetooth: HCI device registration is starting<br>
[   24.447973] HTB: quantum of class 10001 is big. Consider r2q change.<br>
[   24.460334] HTB: quantum of class 10010 is big. Consider r2q change.<br>
[   24.731991] NOHZ: local_softirq_pending 08<br>
[   25.364487] NOHZ: local_softirq_pending 08<br>
[   26.591172] NOHZ: local_softirq_pending 08<br>
[   27.957535] NOHZ: local_softirq_pending 08<br>
[   32.513663] wlan: WCNSS WLAN version 1.5.1.2<br>
[   32.513673] wlan: WCNSS software version CNSS-PR-4-0-00328<br>
[   32.513677] wlan: WCNSS hardware version WCN v2.0 RadioPhy vIris_TSMC_4.0 with 48MHz XO<br>
[   32.513707] DefaultCountry is 00<br>
[   32.528796] wlan_logging_sock_activate_svc: Initalizing FEConsoleLog = 1 NumBuff = 32<br>
[   32.528866] wlan_logging_sock_activate_svc: Initalizing Pkt stats pkt_stats_buff = 16<br>
[   32.529665] send_filled_buffers_to_user: Send Failed -3 drop_count = 0<br>
[   32.530046] wlan: driver loaded<br>
[   32.560073] SMBCHG: wait_for_src_detect: src detect didnt go to a lowered state, still at high, tries = 2, rc = 0<br>
[   32.560109] SMBCHG: rerun_apsd: wait for src detect failed rc = -22<br>
[   32.613146] msm-dwc3 7000000.ssusb: DWC3 exited from low power mode<br>
[   32.680963] enable_store: android_usb: already disabled<br>
[   32.681429] rndis_function_bind_config: rndis_function_bind_config MAC: E6:EE:29:A5:FA:BA<br>
[   32.681475] android_usb gadget: using random self ethernet address<br>
[   32.681491] android_usb gadget: using previous host ethernet address<br>
[   32.682297] rndis0: MAC 06:c1:22:62:de:3a<br>
[   32.682303] rndis0: HOST MAC ee:60:50:97:8f:8c<br>
[   32.682365] hid keyboard<br>
[   32.682374] hidg_bind: creating device ffffffc0840d1200<br>
[   32.682712] hid mouse<br>
[   32.682718] hidg_bind: creating device ffffffc0840d0a00<br>
[   32.689355] IPv6: ADDRCONF(NETDEV_UP): rndis0: link is not ready<br>
[   32.702041] IPv6: ADDRCONF(NETDEV_UP): rndis0: link is not ready<br>
[   32.724110] [21:24:37.606528] [0000000029510524] [VosMC]  wlan: [E :HDD] Ptt Socket error sending message to the app!!<br>
[   32.724230] [21:24:37.606651] [0000000029510E51] [VosMC]  wlan: [E :HDD] Ptt Socket error sending message to the app!!<br>
[   32.725245] [21:24:37.607666] [0000000029515A6F] [VosMC]  wlan: [E :HDD] Ptt Socket error sending message to the app!!<br>
[   32.924099] [21:24:37.806518] [00000000298B9C54] [VosMC]  wlan: [E :HDD] Ptt Socket error sending message to the app!!<br>
[   33.052433] android_work: android_work: sent uevent USB_STATE=CONNECTED<br>
[   33.058907] android_work: android_work: sent uevent USB_STATE=DISCONNECTED<br>
[   33.204362] android_work: android_work: sent uevent USB_STATE=CONNECTED<br>
[   33.237225] android_usb gadget: high-speed config #1: 86000c8.android_usb<br>
[   33.237379] msm-dwc3 7000000.ssusb: Avail curr from USB = 500<br>
[   33.237467] IPv6: ADDRCONF(NETDEV_CHANGE): rndis0: link becomes ready<br>
[   33.292112] android_work: android_work: sent uevent USB_STATE=CONFIGURED<br>
[   34.063036] tz_get_target_freq ADRENO jumping level = 5 last_level = 6 total=12304 busy=12335 original busy_time=12335<br>
[   34.330320] tz_get_target_freq ADRENO jumping level = 6 last_level = 5 total=27313 busy=6314 original busy_time=6314<br>
[   34.455424] tz_get_target_freq ADRENO jumping level = 5 last_level = 6 total=16484 busy=10116 original busy_time=10116<br>
[   50.688345] [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister sit0<br>
[   50.708095] [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister sit0<br>
[   50.712410] [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister lo<br>
[   50.724098] [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister lo<br>
</details><br>

Журнал рабочей Sailfish 3.0.2.8
```console
sh-3.2# journalctl
```
<details>
-- Logs begin at Sat 1970-02-21 22:36:15 EET, end at Tue 2019-03-12 23:28:48 EET. --<br>
Feb 21 22:36:15 Sailfish systemd-journal[686]: Runtime journal (/run/log/journal/) is currently using 4.0M.<br>
                                               Maximum allowed usage is set to 8.0M.<br>
                                               Leaving at least 210.6M free (of currently available 1.3G of space).<br>
                                               Enforced usage limit is thus 8.0M.<br>
Feb 21 22:36:15 Sailfish kernel: Initializing cgroup subsys cpuset<br>
Feb 21 22:36:15 Sailfish kernel: Initializing cgroup subsys cpu<br>
Feb 21 22:36:15 Sailfish kernel: Initializing cgroup subsys cpuacct<br>
Feb 21 22:36:15 Sailfish kernel: Linux version 3.18.105-ElectraBlue-11.0-mido (piggz@linux-f1uu) (gcc version 4.9 20150123 (prerele<br>
ase) (GCC) ) #3 SMP PREEMPT Fri Aug 24 19:52:43 UTC 2018<br>
Feb 21 22:36:15 Sailfish kernel: CPU: AArch64 Processor [410fd034] revision 4<br>
Feb 21 22:36:15 Sailfish kernel: alternatives: enabling workaround for ARM erratum 845719<br>
Feb 21 22:36:15 Sailfish kernel: Machine: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3<br>
Feb 21 22:36:15 Sailfish kernel: efi: Getting EFI parameters from FDT:<br>
Feb 21 22:36:15 Sailfish kernel: efi: UEFI not found.<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: reserved region for node 'other_ext_region@0': base 0x0000000084a00000, size 30 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: reserved region for node 'modem_region@0': base 0x0000000086c00000, size 106 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: reserved region for node 'adsp_fw_region@0': base 0x000000008d600000, size 17 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: reserved region for node 'wcnss_fw_region@0': base 0x000000008e700000, size 7 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: reserved region for node 'dfps_data_mem@90000000': base 0x0000000090000000, size 0 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: reserved region for node 'splash_region@0x90001000': base 0x0000000090001000, size 19 M<br>
iB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: reserved region for node 'pstore_reserve_mem_region@0': base 0x000000009ff00000, size 1<br>
 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Removed memory: created DMA memory pool at 0x0000000084a00000, size 30 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: initialized node other_ext_region@0, compatible id removed-dma-pool<br>
Feb 21 22:36:15 Sailfish kernel: Removed memory: created DMA memory pool at 0x0000000086c00000, size 106 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: initialized node modem_region@0, compatible id removed-dma-pool<br>
Feb 21 22:36:15 Sailfish kernel: Removed memory: created DMA memory pool at 0x000000008d600000, size 17 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: initialized node adsp_fw_region@0, compatible id removed-dma-pool<br>
Feb 21 22:36:15 Sailfish kernel: Removed memory: created DMA memory pool at 0x000000008e700000, size 7 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: initialized node wcnss_fw_region@0, compatible id removed-dma-pool<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: allocated memory for 'venus_region@0' node: base 0x000000008f800000, size 8 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: created CMA memory pool at 0x000000008f800000, size 8 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: initialized node venus_region@0, compatible id shared-dma-pool<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: allocated memory for 'secure_region@0' node: base 0x00000000f6800000, size 152 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: created CMA memory pool at 0x00000000f6800000, size 152 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: initialized node secure_region@0, compatible id shared-dma-pool<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: allocated memory for 'qseecom_region@0' node: base 0x00000000f5800000, size 16 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: created CMA memory pool at 0x00000000f5800000, size 16 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: initialized node qseecom_region@0, compatible id shared-dma-pool<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: allocated memory for 'adsp_region@0' node: base 0x00000000f5400000, size 4 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: created CMA memory pool at 0x00000000f5400000, size 4 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: initialized node adsp_region@0, compatible id shared-dma-pool<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: allocated memory for 'gpu_region@0' node: base 0x000000008f000000, size 8 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: created CMA memory pool at 0x000000008f000000, size 8 MiB<br>
Feb 21 22:36:15 Sailfish kernel: Reserved memory: initialized node gpu_region@0, compatible id shared-dma-pool<br>
Feb 21 22:36:15 Sailfish kernel: cma: Reserved 16 MiB at 0x00000000f4400000<br>
Feb 21 22:36:15 Sailfish kernel: On node 0 totalpages: 772608<br>
Feb 21 22:36:15 Sailfish kernel:   DMA zone: 12288 pages used for memmap<br>
Feb 21 22:36:15 Sailfish kernel:   DMA zone: 0 pages reserved<br>
Feb 21 22:36:15 Sailfish kernel:   DMA zone: 772608 pages, LIFO batch:31<br>
Feb 21 22:36:15 Sailfish kernel: psci: probing for conduit method from DT.<br>
Feb 21 22:36:15 Sailfish kernel: psci: PSCIv1.0 detected in firmware.<br>
Feb 21 22:36:15 Sailfish kernel: psci: Using standard PSCI v0.2 function IDs<br>
Feb 21 22:36:15 Sailfish kernel: psci: Initializing psci_cpu_init<br>
Feb 21 22:36:15 Sailfish kernel: psci: Initializing psci_cpu_init<br>
Feb 21 22:36:15 Sailfish kernel: psci: Initializing psci_cpu_init<br>
Feb 21 22:36:15 Sailfish kernel: psci: Initializing psci_cpu_init<br>
Feb 21 22:36:15 Sailfish kernel: psci: Initializing psci_cpu_init<br>
Feb 21 22:36:15 Sailfish kernel: psci: Initializing psci_cpu_init<br>
Feb 21 22:36:15 Sailfish kernel: psci: Initializing psci_cpu_init<br>
Feb 21 22:36:15 Sailfish kernel: PERCPU: Embedded 15 pages/cpu @ffffffc0b418e000 s22912 r8192 d30336 u61440<br>
Feb 21 22:36:15 Sailfish kernel: pcpu-alloc: s22912 r8192 d30336 u61440 alloc=15*4096<br>
Feb 21 22:36:15 Sailfish kernel: pcpu-alloc: [0] 0 [0] 1 [0] 2 [0] 3 [0] 4 [0] 5 [0] 6 [0] 7<br>
Feb 21 22:36:15 Sailfish kernel: Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 760320<br>
Feb 21 22:36:15 Sailfish kernel: Kernel command line: boot_cpus=0,1,2,3,4,5,6,7 sched_enable_hmp=1 sched_enable_power_aware=1 andro<br>
idboot.hardware=qcom msm_rtb.filter=0x237 ehci-hcd.park=3 lpm_levels.sleep_disabled=1 androidboot.bootdevice=7824900.sdhci earlycon=msm_h<br>
sl_uart,0x78af000 androidboot.emmc=true androidboot.verifiedbootstate=orange androidboot.veritymode=enforcing androidboot.keymaster=1 and<br>
roidboot.serialno=6fcdaa929904 androidboot.boot_reason= androidboot.secureboot=1 androidboot.baseband=msm mdss_mdp.panel=1:dsi:0:qcom,mds<br>
s_dsi_nt35532_fhd_video:1:none:cfg:single_dsi<br>
Feb 21 22:36:15 Sailfish kernel: zakk: booting_into_recovery=0<br>
Feb 21 22:36:15 Sailfish kernel: PID hash table entries: 4096 (order: 3, 32768 bytes)<br>
Feb 21 22:36:15 Sailfish kernel: Dentry cache hash table entries: 524288 (order: 10, 4194304 bytes)<br>
Feb 21 22:36:15 Sailfish kernel: Inode-cache hash table entries: 262144 (order: 9, 2097152 bytes)<br>
Feb 21 22:36:15 Sailfish kernel: Memory: 2665368K/3090432K available (13969K kernel code, 1848K rwdata, 6996K rodata, 892K init, 32<br>
56K bss, 216168K reserved, 208896K cma-reserved)<br>
Feb 21 22:36:15 Sailfish kernel: Virtual kernel memory layout:<br>
                                     vmalloc : 0xffffff8000000000 - 0xffffffbdbfff0000   (   246 GB)<br>
                                     vmemmap : 0xffffffbdc0000000 - 0xffffffbfc0000000   (     8 GB maximum)<br>
                                               0xffffffbdc0000000 - 0xffffffbdc3000000   (    48 MB actual)<br>
                                     PCI I/O : 0xffffffbffa000000 - 0xffffffbffb000000   (    16 MB)<br>
                                     fixed   : 0xffffffbffbdfd000 - 0xffffffbffbdff000   (     8 KB)<br>
                                     modules : 0xffffffbffc000000 - 0xffffffc000000000   (    64 MB)<br>
                                     memory  : 0xffffffc000000000 - 0xffffffc0c0000000   (  3072 MB)<br>
                                       .init : 0xffffffc0014fc000 - 0xffffffc0015db000   (   892 KB)<br>
                                       .text : 0xffffffc000080000 - 0xffffffc0014fb474   ( 20974 KB)<br>
                                       .data : 0xffffffc0015f9000 - 0xffffffc0017c7000   (  1848 KB)<br>
Feb 21 22:36:15 Sailfish kernel: SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=8, Nodes=1<br>
Feb 21 22:36:15 Sailfish kernel: HMP scheduling enabled.<br>
Feb 21 22:36:15 Sailfish kernel: Preemptible hierarchical RCU implementation.<br>
Feb 21 22:36:15 Sailfish kernel:         RCU dyntick-idle grace-period acceleration is enabled.<br>
Feb 21 22:36:15 Sailfish kernel: NR_IRQS:64 nr_irqs:64 0<br>
Feb 21 22:36:15 Sailfish kernel: mpm_init_irq_domain(): Cannot find irq controller for qcom,gpio-parent<br>
Feb 21 22:36:15 Sailfish kernel: MPM 1 irq mapping errored -517<br>
Feb 21 22:36:15 Sailfish kernel:         Offload RCU callbacks from all CPUs<br>
Feb 21 22:36:15 Sailfish kernel:         Offload RCU callbacks from CPUs: 0-7.<br>
Feb 21 22:36:15 Sailfish kernel: Architected cp15 and mmio timer(s) running at 19.20MHz (virt/virt).<br>
Feb 21 22:36:15 Sailfish kernel: sched_clock: 56 bits at 19MHz, resolution 52ns, wraps every 3579139424256ns<br>
Feb 21 22:36:15 Sailfish kernel: Switched to clocksource arch_sys_counter<br>
Feb 21 22:36:15 Sailfish kernel: Console: colour dummy device 80x25<br>
Feb 21 22:36:15 Sailfish kernel: console [tty0] enabled<br>
Feb 21 22:36:15 Sailfish kernel: allocated 12582912 bytes of page_cgroup<br>
Feb 21 22:36:15 Sailfish kernel: please try 'cgroup_disable=memory' option if you don't want memory cgroups<br>
Feb 21 22:36:15 Sailfish kernel: Calibrating delay loop (skipped), value calculated using timer frequency.. 38.40 BogoMIPS (lpj=76800)<br>
Feb 21 22:36:15 Sailfish kernel: pid_max: default: 32768 minimum: 301<br>
Feb 21 22:36:15 Sailfish kernel: Security Framework initialized<br>
Feb 21 22:36:15 Sailfish kernel: SELinux:  Initializing.<br>
Feb 21 22:36:15 Sailfish kernel: SELinux:  Starting in permissive mode<br>
Feb 21 22:36:15 Sailfish kernel: Mount-cache hash table entries: 8192 (order: 4, 65536 bytes)<br>
Feb 21 22:36:15 Sailfish kernel: Mountpoint-cache hash table entries: 8192 (order: 4, 65536 bytes)<br>
Feb 21 22:36:15 Sailfish kernel: Initializing cgroup subsys memory<br>
Feb 21 22:36:15 Sailfish kernel: Initializing cgroup subsys devices<br>
Feb 21 22:36:15 Sailfish kernel: Initializing cgroup subsys freezer<br>
Feb 21 22:36:15 Sailfish kernel: Initializing cgroup subsys net_cls<br>
Feb 21 22:36:15 Sailfish kernel: Initializing cgroup subsys blkio<br>
Feb 21 22:36:15 Sailfish kernel: Initializing cgroup subsys perf_event<br>
Feb 21 22:36:15 Sailfish kernel: ftrace: allocating 45612 entries in 179 pages<br>
Feb 21 22:36:15 Sailfish kernel: /cpus/cpu@0: Missing clock-frequency property<br>
Feb 21 22:36:15 Sailfish kernel: /cpus/cpu@1: Missing clock-frequency property<br>
Feb 21 22:36:15 Sailfish kernel: /cpus/cpu@2: Missing clock-frequency property<br>
Feb 21 22:36:15 Sailfish kernel: /cpus/cpu@3: Missing clock-frequency property<br>
Feb 21 22:36:15 Sailfish kernel: /cpus/cpu@100: Missing clock-frequency property<br>
Feb 21 22:36:15 Sailfish kernel: /cpus/cpu@101: Missing clock-frequency property<br>
Feb 21 22:36:15 Sailfish kernel: /cpus/cpu@102: Missing clock-frequency property<br>
Feb 21 22:36:15 Sailfish kernel: /cpus/cpu@103: Missing clock-frequency property<br>
Feb 21 22:36:15 Sailfish kernel: hw perfevents: enabled with arm/armv8-pmuv3 PMU driver, 7 counters available<br>
Feb 21 22:36:15 Sailfish kernel: EFI services will not be available.<br>
Feb 21 22:36:15 Sailfish kernel: NOHZ: local_softirq_pending 02<br>
Feb 21 22:36:15 Sailfish kernel: MSM Memory Dump base table set up<br>
Feb 21 22:36:15 Sailfish kernel: MSM Memory Dump apps data table set up<br>
Feb 21 22:36:15 Sailfish kernel: Configuring XPU violations to be fatal errors<br>
Feb 21 22:36:15 Sailfish kernel: cpu_clock_pwr_init: Power clocks configured<br>
Feb 21 22:36:15 Sailfish kernel: Hotplug policy switch err. Task swapper/0 pid=1<br>
Feb 21 22:36:15 Sailfish kernel: Hotplug policy switch err. Task swapper/0 pid=1<br>
Feb 21 22:36:15 Sailfish kernel: Hotplug policy switch err. Task swapper/0 pid=1<br>
Feb 21 22:36:15 Sailfish kernel: Hotplug policy switch err. Task swapper/0 pid=1<br>
Feb 21 22:36:15 Sailfish kernel: Hotplug policy switch err. Task swapper/0 pid=1<br>
Feb 21 22:36:15 Sailfish kernel: Hotplug policy switch err. Task swapper/0 pid=1<br>
Feb 21 22:36:15 Sailfish kernel: Hotplug policy switch err. Task swapper/0 pid=1<br>
Feb 21 22:36:15 Sailfish kernel: Brought up 8 CPUs<br>
Feb 21 22:36:15 Sailfish kernel: SMP: Total of 8 processors activated.<br>
Feb 21 22:36:15 Sailfish kernel: alternatives: patching kernel code<br>
Feb 21 22:36:15 Sailfish kernel: devtmpfs: initialized<br>
Feb 21 22:36:15 Sailfish kernel: DMI not present or invalid.<br>
Feb 21 22:36:15 Sailfish kernel: pinctrl core: initialized pinctrl subsystem<br>
Feb 21 22:36:15 Sailfish kernel: regulator-dummy: no parameters<br>
Feb 21 22:36:15 Sailfish kernel: NET: Registered protocol family 16<br>
Feb 21 22:36:15 Sailfish kernel: ramoops: using module parameters<br>
Feb 21 22:36:15 Sailfish kernel: persistent_ram: found existing invalid buffer, size 104399, start 104399<br>
Feb 21 22:36:15 Sailfish kernel: console [pstore-1] enabled<br>
Feb 21 22:36:15 Sailfish kernel: pstore: Registered ramoops as persistent store backend<br>
Feb 21 22:36:15 Sailfish kernel: ramoops: attached 0x100000@0x9ff00000, ecc: 0/0<br>
Feb 21 22:36:15 Sailfish kernel: ------------[ cut here ]------------<br>
Feb 21 22:36:15 Sailfish kernel: WARNING: CPU: 2 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi<br>
/msm8953/drivers/power/qcom/pm-boot.c:45 msm_pm_boot_init+0x90/0xd0()<br>
Feb 21 22:36:15 Sailfish kernel: Modules linked in:<br>
Feb 21 22:36:15 Sailfish kernel: CPU: 2 PID: 1 Comm: swapper/0 Not tainted 3.18.105-ElectraBlue-11.0-mido #3<br>
Feb 21 22:36:15 Sailfish kernel: Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
Feb 21 22:36:15 Sailfish kernel: Call trace:<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a2c4] show_stack+0x20/0x28<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a69ac] warn_slowpath_null+0x38/0x44<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc001541ad4] msm_pm_boot_init+0x90/0xd0<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e03928] kernel_init+0x20/0xe0<br>
Feb 21 22:36:15 Sailfish kernel: ---[ end trace baf5d4897624fa07 ]---<br>
Feb 21 22:36:15 Sailfish kernel: scm_call failed: func id 0x42000101, ret: -1, syscall returns: 0x0, 0x0, 0x0<br>
Feb 21 22:36:15 Sailfish kernel: cpuidle: using governor ladder<br>
Feb 21 22:36:15 Sailfish kernel: cpuidle: using governor menu<br>
Feb 21 22:36:15 Sailfish kernel: cpuidle: using governor qcom<br>
Feb 21 22:36:15 Sailfish kernel: vdso: 2 pages (1 code @ ffffffc001601000, 1 data @ ffffffc001600000)<br>
Feb 21 22:36:15 Sailfish kernel: software IO TLB [mem 0xeec00000-0xef000000] (4MB) mapped at [ffffffc0aec00000-ffffffc0aeffffff]<br>
Feb 21 22:36:15 Sailfish kernel: DMA: preallocated 256 KiB pool for atomic allocations<br>
Feb 21 22:36:15 Sailfish kernel: __of_mpm_init(): MPM driver mapping exists<br>
Feb 21 22:36:15 Sailfish kernel: platform soc:qcom,kgsl-hyp: assigned reserved memory node gpu_region@0<br>
Feb 21 22:36:15 Sailfish kernel: msm_mpm_dev_probe(): Cannot get clk resource for XO: -517<br>
Feb 21 22:36:15 Sailfish kernel: sps:sps is ready.<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:vdd_restriction_reg_init Defer regulator vdd-dig probe<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:probe_vdd_rstr Err regulator init. err:-517. KTM continues.<br>
Feb 21 22:36:15 Sailfish kernel: msm-thermal soc:qcom,msm-thermal: probe_vdd_rstr:Failed reading node=/soc/qcom,msm-thermal, key=qcom,max<br>
-freq-level. err=-517. KTM continues<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:msm_thermal_dev_probe Failed reading node=/soc/qcom,msm-thermal, key=qcom,online-hotpl<br>
ug-core. err:-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b6000.i2c: probing driver i2c-msm-v2<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b6000.i2c: error on clk_get(core_clk):-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b6000.i2c: error probe() failed with err:-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b7000.i2c: probing driver i2c-msm-v2<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b7000.i2c: error on clk_get(core_clk):-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b7000.i2c: error probe() failed with err:-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 7af5000.i2c: probing driver i2c-msm-v2<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 7af5000.i2c: error on clk_get(core_clk):-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 7af5000.i2c: error probe() failed with err:-517<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: Speed bin: 2 PVS Version: 0<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: Get vdd-mx regulator!!!<br>
Feb 21 22:36:15 Sailfish kernel: msm_rpm_glink_dt_parse: qcom,rpm-glink compatible not matches<br>
Feb 21 22:36:15 Sailfish kernel: msm_rpm_dev_probe: APSS-RPM communication over SMD<br>
Feb 21 22:36:15 Sailfish kernel: smd_channel_probe_now: allocation table not initialized<br>
Feb 21 22:36:15 Sailfish kernel: smd_channel_probe_now: allocation table not initialized<br>
Feb 21 22:36:15 Sailfish kernel: smd_channel_probe_now: allocation table not initialized<br>
Feb 21 22:36:15 Sailfish kernel: msm_watchdog b017000.qcom,wdt: wdog absent resource not present<br>
Feb 21 22:36:15 Sailfish kernel: msm_watchdog b017000.qcom,wdt: MSM Watchdog Initialized<br>
Feb 21 22:36:15 Sailfish kernel: msm-sn-fuse a4128.snfuse: SN interface initialized<br>
Feb 21 22:36:15 Sailfish kernel: platform soc:qcom,adsprpc-mem: assigned reserved memory node adsp_region@0<br>
Feb 21 22:36:15 Sailfish kernel: spmi_pmic_arb 200f000.qcom,spmi: PMIC Arb Version-2 0x20010000<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_s5: 400 -- 1140 mV at 870 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_s5_avs_limit: 400 -- 1140 mV<br>
Feb 21 22:36:15 Sailfish kernel: spm_regulator_probe: name=pm8953_s5, range=LV, voltage=870000 uV, mode=AUTO, step rate=1200 uV/us<br>
Feb 21 22:36:15 Sailfish kernel: platform 4080000.qcom,mss: assigned reserved memory node modem_region@0<br>
Feb 21 22:36:15 Sailfish kernel: platform c200000.qcom,lpass: assigned reserved memory node adsp_fw_region@0<br>
Feb 21 22:36:15 Sailfish kernel: platform 1de0000.qcom,venus: assigned reserved memory node venus_region@0<br>
Feb 21 22:36:15 Sailfish kernel: platform a21b000.qcom,pronto: assigned reserved memory node wcnss_fw_region@0<br>
Feb 21 22:36:15 Sailfish kernel: apc_mem_acc_corner: 0 -- 0 mV<br>
Feb 21 22:36:15 Sailfish kernel: cpr4_msm8953_apss_read_fuse_data: apc_corner: speed bin = 2<br>
Feb 21 22:36:15 Sailfish kernel: cpr4_msm8953_apss_read_fuse_data: apc_corner: CPR fusing revision = 3<br>
Feb 21 22:36:15 Sailfish kernel: cpr4_msm8953_apss_read_fuse_data: apc_corner: foundry id = 2<br>
Feb 21 22:36:15 Sailfish kernel: cpr4_msm8953_apss_read_fuse_data: apc_corner: CPR misc fuse value = 0<br>
Feb 21 22:36:15 Sailfish kernel: cpr4_msm8953_apss_read_fuse_data: apc_corner: Voltage boost fuse config = 0 boost = disable<br>
Feb 21 22:36:15 Sailfish kernel: cpr4_msm8953_apss_calculate_open_loop_voltages: apc_corner: fused   LowSVS: open-loop= 625000 uV<br>
Feb 21 22:36:15 Sailfish kernel: cpr4_msm8953_apss_calculate_open_loop_voltages: apc_corner: fused      SVS: open-loop= 690000 uV<br>
Feb 21 22:36:15 Sailfish kernel: cpr4_msm8953_apss_calculate_open_loop_voltages: apc_corner: fused      NOM: open-loop= 805000 uV<br>
Feb 21 22:36:15 Sailfish kernel: cpr4_msm8953_apss_calculate_open_loop_voltages: apc_corner: fused TURBO_L1: open-loop= 915000 uV<br>
Feb 21 22:36:15 Sailfish kernel: cpr4_msm8953_apss_calculate_target_quotients: apc_corner: fused   LowSVS: quot[ 7]= 441<br>
Feb 21 22:36:15 Sailfish kernel: cpr4_msm8953_apss_calculate_target_quotients: apc_corner: fused      SVS: quot[ 7]= 550, quot_offset[ 7]<br>
= 105<br>
Feb 21 22:36:15 Sailfish kernel: cpr4_msm8953_apss_calculate_target_quotients: apc_corner: fused      NOM: quot[ 7]= 788, quot_offset[ 7]<br>
= 235<br>
Feb 21 22:36:15 Sailfish kernel: cpr4_msm8953_apss_calculate_target_quotients: apc_corner: fused TURBO_L1: quot[ 7]=1009, quot_offset[ 7]<br>
= 220<br>
Feb 21 22:36:15 Sailfish kernel: cpr4_apss_init_aging: apc: sensor 6 aging init quotient diff = 5, aging RO scale = 2800 QUOT/V<br>
Feb 21 22:36:15 Sailfish kernel: cpr3_regulator_init_ctrl: apc: Default CPR mode = HW closed-loop<br>
Feb 21 22:36:15 Sailfish kernel: apc_corner: 0 -- 0 mV at 0 mV<br>
Feb 21 22:36:15 Sailfish kernel: gfx_mem_acc_corner: 0 -- 0 mV<br>
Feb 21 22:36:15 Sailfish kernel: GFX_LDO: msm_gfx_ldo_parse_dt: Unable to parse CX parameters rc=-517<br>
Feb 21 22:36:15 Sailfish kernel: GFX_LDO: msm_gfx_ldo_probe: Unable to pasrse dt rc=-517<br>
Feb 21 22:36:15 Sailfish kernel: msm_mpm_dev_probe(): Cannot get clk resource for XO: -517<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:sensor_mgr_init_threshold threshold id already initialized<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:vdd_restriction_reg_init Defer regulator vdd-dig probe<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:probe_vdd_rstr Err regulator init. err:-517. KTM continues.<br>
Feb 21 22:36:15 Sailfish kernel: msm-thermal soc:qcom,msm-thermal: probe_vdd_rstr:Failed reading node=/soc/qcom,msm-thermal, key=qcom,max<br>
-freq-level. err=-517. KTM continues<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:msm_thermal_dev_probe Failed reading node=/soc/qcom,msm-thermal, key=qcom,online-hotpl<br>
ug-core. err:-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b6000.i2c: probing driver i2c-msm-v2<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b6000.i2c: error on clk_get(core_clk):-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b6000.i2c: error probe() failed with err:-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b7000.i2c: probing driver i2c-msm-v2<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b7000.i2c: error on clk_get(core_clk):-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b7000.i2c: error probe() failed with err:-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 7af5000.i2c: probing driver i2c-msm-v2<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 7af5000.i2c: error on clk_get(core_clk):-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 7af5000.i2c: error probe() failed with err:-517<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: Speed bin: 2 PVS Version: 0<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: Get vdd-mx regulator!!!<br>
Feb 21 22:36:15 Sailfish kernel: msm_rpm_glink_dt_parse: qcom,rpm-glink compatible not matches<br>
Feb 21 22:36:15 Sailfish kernel: msm_rpm_dev_probe: APSS-RPM communication over SMD<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_s1: 870 -- 1156 mV at 1000 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_s2_level: 0 -- 0 mV at 0 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_s2_floor_level: 0 -- 0 mV at 0 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_s2_level_ao: 0 -- 0 mV at 0 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_s3: 1225 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_s4: 1900 -- 2050 mV at 1900 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_s7_level: 0 -- 0 mV at 0 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_s7_level_ao: 0 -- 0 mV at 0 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_s7_level_so: 0 -- 0 mV at 0 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l1: 1000 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l2: 975 -- 1225 mV at 975 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l3: 925 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l5: 1800 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l6: 1800 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l7: 1800 -- 1900 mV at 1800 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l7_ao: 1800 -- 1900 mV at 1800 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l8: 2900 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l9: 3000 -- 3300 mV at 3000 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l10: 2850 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l11: 2950 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l12: 1800 -- 2950 mV at 1800 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l13: 3125 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l16: 1800 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l17: 2850 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l19: 1200 -- 1350 mV at 1200 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l22: 2800 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: pm8953_l23: 975 -- 1225 mV at 975 mV normal idle<br>
Feb 21 22:36:15 Sailfish kernel: GFX_LDO: msm_gfx_ldo_voltage_init: LDO corner 1: target-volt = 580000 uV<br>
Feb 21 22:36:15 Sailfish kernel: GFX_LDO: msm_gfx_ldo_voltage_init: LDO corner 2: target-volt = 650000 uV<br>
Feb 21 22:36:15 Sailfish kernel: GFX_LDO: msm_gfx_ldo_voltage_init: LDO corner 3: target-volt = 720000 uV<br>
Feb 21 22:36:15 Sailfish kernel: GFX_LDO: msm_gfx_ldo_adjust_init_voltage: adjusted the open-loop voltage[1] 580000 - 615000<br>
Feb 21 22:36:15 Sailfish kernel: GFX_LDO: msm_gfx_ldo_adjust_init_voltage: adjusted the open-loop voltage[2] 650000 - 675000<br>
Feb 21 22:36:15 Sailfish kernel: GFX_LDO: msm_gfx_ldo_voltage_init: LDO-mode fuse disabled by default<br>
Feb 21 22:36:15 Sailfish kernel: msm_gfx_ldo: 0 -- 0 mV at 0 mV<br>
Feb 21 22:36:15 Sailfish kernel: msm_mpm_dev_probe(): Cannot get clk resource for XO: -517<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:sensor_mgr_init_threshold threshold id already initialized<br>
Feb 21 22:36:15 Sailfish kernel: cpu cpu0: dev_pm_opp_get_opp_count: device OPP not found (-19)<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:get_cpu_freq_plan_len Error reading CPU0 freq table len. error:-19<br>
Feb 21 22:36:15 Sailfish kernel: cpu cpu4: dev_pm_opp_get_opp_count: device OPP not found (-19)<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:get_cpu_freq_plan_len Error reading CPU4 freq table len. error:-19<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:vdd_restriction_reg_init Defer vdd rstr freq init.<br>
Feb 21 22:36:15 Sailfish kernel: cpu cpu0: dev_pm_opp_get_opp_count: device OPP not found (-19)<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:get_cpu_freq_plan_len Error reading CPU0 freq table len. error:-19<br>
Feb 21 22:36:15 Sailfish kernel: cpu cpu4: dev_pm_opp_get_opp_count: device OPP not found (-19)<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:get_cpu_freq_plan_len Error reading CPU4 freq table len. error:-19<br>
Feb 21 22:36:15 Sailfish kernel: cpu cpu0: dev_pm_opp_get_opp_count: device OPP not found (-19)<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:get_cpu_freq_plan_len Error reading CPU0 freq table len. error:-19<br>
Feb 21 22:36:15 Sailfish kernel: cpu cpu4: dev_pm_opp_get_opp_count: device OPP not found (-19)<br>
Feb 21 22:36:15 Sailfish kernel: msm_thermal:get_cpu_freq_plan_len Error reading CPU4 freq table len. error:-19<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b6000.i2c: probing driver i2c-msm-v2<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b6000.i2c: error on clk_get(core_clk):-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b6000.i2c: error probe() failed with err:-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b7000.i2c: probing driver i2c-msm-v2<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b7000.i2c: error on clk_get(core_clk):-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b7000.i2c: error probe() failed with err:-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 7af5000.i2c: probing driver i2c-msm-v2<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 7af5000.i2c: error on clk_get(core_clk):-517<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 7af5000.i2c: error probe() failed with err:-517<br>
Feb 21 22:36:15 Sailfish kernel: ------------[ cut here ]------------<br>
Feb 21 22:36:15 Sailfish kernel: WARNING: CPU: 3 PID: 6 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi<br>
/msm8953/drivers/clk/msm/clock-local2.c:1005 branch_clk_handoff+0x7c/0xc0()<br>
Feb 21 22:36:15 Sailfish kernel: gcc_usb_phy_cfg_ahb_clk clock is enabled in HW even though ENABLE_BIT is not set<br>
Feb 21 22:36:15 Sailfish kernel: Modules linked in:<br>
Feb 21 22:36:15 Sailfish kernel: CPU: 3 PID: 6 Comm: kworker/u16:0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
Feb 21 22:36:15 Sailfish kernel: Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
Feb 21 22:36:15 Sailfish kernel: Workqueue: deferwq deferred_probe_work_func<br>
Feb 21 22:36:15 Sailfish kernel: Call trace:<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a2c4] show_stack+0x20/0x28<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000aefe1c] branch_clk_handoff+0x7c/0xc0<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000aec2ec] __handoff_clk+0xd0/0x2e4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000aec684] msm_clock_register+0x184/0x248<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000aec7d8] of_msm_clock_register+0x90/0xa0<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000afceb4] msm_gcc_probe+0x120/0x214<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000651580] platform_drv_probe+0x44/0x90<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00064f628] driver_probe_device+0xe0/0x22c<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00064f7a8] __device_attach+0x34/0x54<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00064d7ec] bus_for_each_drv+0x98/0xc8<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00064f51c] device_attach+0x78/0xa4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00064ea5c] bus_probe_device+0x38/0xac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00064ef18] deferred_probe_work_func+0x7c/0xa8<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000bf69c] process_one_work+0x25c/0x438<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000c00c8] worker_thread+0x32c/0x448<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000c4aa8] kthread+0xf8/0x100<br>
Feb 21 22:36:15 Sailfish kernel: ---[ end trace baf5d4897624fa08 ]---<br>
Feb 21 22:36:15 Sailfish kernel: qcom,gcc-8953 1800000.qcom,gcc: Registered GCC clocks<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: Speed bin: 2 PVS Version: 0<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: missing qcom,dfs-sid-c0<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: No DFS SID tables found for Cluster-0<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: missing qcom,link-sid-c0<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: No Link SID tables found for Cluster-0<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: missing qcom,lmh-sid-c0<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: No LMH SID tables found for Cluster-0<br>
Feb 21 22:36:15 Sailfish kernel: ramp_lmh_sid: Use Default LMH SID<br>
Feb 21 22:36:15 Sailfish kernel: ramp_dfs_sid: Use Default DFS SID<br>
Feb 21 22:36:15 Sailfish kernel: ramp_link_sid: Use Default Link SID<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: missing qcom,dfs-sid-c1<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: No DFS SID tables found for Cluster-1<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: missing qcom,link-sid-c1<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: No Link SID tables found for Cluster-1<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: missing qcom,lmh-sid-c1<br>
Feb 21 22:36:15 Sailfish kernel: cpu-clock-8953 b114000.qcom,cpu-clock-8953: No LMH SID tables found for Cluster-1<br>
Feb 21 22:36:15 Sailfish kernel: ramp_lmh_sid: Use Default LMH SID<br>
Feb 21 22:36:15 Sailfish kernel: ramp_dfs_sid: Use Default DFS SID<br>
Feb 21 22:36:15 Sailfish kernel: ramp_link_sid: Use Default Link SID<br>
Feb 21 22:36:15 Sailfish kernel: clock_rcgwr_init: RCGwR  Init Completed<br>
Feb 21 22:36:15 Sailfish kernel: populate_opp_table: clock-cpu-8953: OPP tables populated (cpu 3 and 7)<br>
Feb 21 22:36:15 Sailfish kernel: print_opp_table: clock_cpu: a53 C0: OPP voltage for 652800000: 1<br>
Feb 21 22:36:15 Sailfish kernel: print_opp_table: clock_cpu: a53 C0: OPP voltage for 2016000000: 7<br>
Feb 21 22:36:15 Sailfish kernel: print_opp_table: clock_cpu: a53 C1: OPP voltage for 652800000: 1<br>
Feb 21 22:36:15 Sailfish kernel: print_opp_table: clock_cpu: a53 C2: OPP voltage for 2016000000: 7<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b6000.i2c: probing driver i2c-msm-v2<br>
Feb 21 22:36:15 Sailfish kernel: AXI: msm_bus_scale_register_client(): msm_bus_scale_register_client: Bus driver not ready.<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b6000.i2c: msm_bus_scale_register_client(mstr-id:86):0 (not a problem)<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b7000.i2c: probing driver i2c-msm-v2<br>
Feb 21 22:36:15 Sailfish kernel: AXI: msm_bus_scale_register_client(): msm_bus_scale_register_client: Bus driver not ready.<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b7000.i2c: msm_bus_scale_register_client(mstr-id:86):0 (not a problem)<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b7000.i2c: irq:176 when no active transfer<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 7af5000.i2c: probing driver i2c-msm-v2<br>
Feb 21 22:36:15 Sailfish kernel: AXI: msm_bus_scale_register_client(): msm_bus_scale_register_client: Bus driver not ready.<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 7af5000.i2c: msm_bus_scale_register_client(mstr-id:84):0 (not a problem)<br>
Feb 21 22:36:15 Sailfish kernel: gcc-gfx-8953 1800000.qcom,gcc-gfx: Registered GCC GFX clocks.<br>
Feb 21 22:36:15 Sailfish kernel: qcom,qpnp-pin qpnp-pin-17: qpnp_pin_probe: no device nodes specified in topology<br>
Feb 21 22:36:15 Sailfish kernel: qcom,qpnp-pin: probe of qpnp-pin-17 failed with error -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_cpuss_dump soc:cpuss_dump: Couldn't get memory for dumping<br>
Feb 21 22:36:15 Sailfish kernel: msm_cpuss_dump soc:cpuss_dump: Couldn't get memory for dumping<br>
Feb 21 22:36:15 Sailfish kernel: KPI: Bootloader start count = 28025<br>
Feb 21 22:36:15 Sailfish kernel: KPI: Bootloader end count = 101763<br>
Feb 21 22:36:15 Sailfish kernel: KPI: Bootloader display count = 72449<br>
Feb 21 22:36:15 Sailfish kernel: KPI: Bootloader load kernel count = 1282<br>
Feb 21 22:36:15 Sailfish kernel: KPI: Kernel MPM timestamp = 129004<br>
Feb 21 22:36:15 Sailfish kernel: KPI: Kernel MPM Clock frequency = 32768<br>
Feb 21 22:36:15 Sailfish kernel: socinfo_print: v0.10, id=293, ver=1.1, raw_id=70, raw_ver=1, hw_plat=11, hw_plat_ver=65536<br>
                                  accessory_chip=0, hw_plat_subtype=0, pmic_model=65558, pmic_die_revision=65536 foundry_id=3 serial_numb<br>
er=1599825137<br>
Feb 21 22:36:15 Sailfish kernel: eldo2_8953: no parameters<br>
Feb 21 22:36:15 Sailfish kernel: qpnp_pin_of_gpio_xlate: no such PMIC gpio 5 in device topology<br>
Feb 21 22:36:15 Sailfish kernel: adv_vreg: no parameters<br>
Feb 21 22:36:15 Sailfish kernel: vdd_vreg: no parameters<br>
Feb 21 22:36:15 Sailfish kernel: vgaarb: loaded<br>
Feb 21 22:36:15 Sailfish kernel: SCSI subsystem initialized<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver usbfs<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver hub<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new device driver usb<br>
Feb 21 22:36:15 Sailfish kernel: media: Linux media interface: v0.10<br>
Feb 21 22:36:15 Sailfish kernel: Linux video capture interface: v2.00<br>
Feb 21 22:36:15 Sailfish kernel: EDAC MC: Ver: 3.0.0<br>
Feb 21 22:36:15 Sailfish kernel: cpufreq: driver msm up and running<br>
Feb 21 22:36:15 Sailfish kernel: platform soc:qcom,ion:qcom,ion-heap@8: assigned reserved memory node secure_region@0<br>
Feb 21 22:36:15 Sailfish kernel: platform soc:qcom,ion:qcom,ion-heap@27: assigned reserved memory node qseecom_region@0<br>
Feb 21 22:36:15 Sailfish kernel: ION heap system created<br>
Feb 21 22:36:15 Sailfish kernel: ION heap mm created at 0x00000000f6800000 with size 9800000<br>
Feb 21 22:36:15 Sailfish kernel: ION heap qsecom created at 0x00000000f5800000 with size 1000000<br>
Feb 21 22:36:15 Sailfish kernel: msm_bus_fabric_init_driver<br>
Feb 21 22:36:15 Sailfish kernel: msm_bus_device 580000.ad-hoc-bus: Coresight support absent for bus: 2048<br>
Feb 21 22:36:15 Sailfish kernel: qcom,qpnp-power-on qpnp-power-on-1: PMIC@SID0 Power-on reason: Triggered from Hard Reset and 'warm' boot<br>
Feb 21 22:36:15 Sailfish kernel: qcom,qpnp-power-on qpnp-power-on-1: PMIC@SID0: Power-off reason: Triggered from KPDPWR_N (Long Power Key<br>
 hold)<br>
Feb 21 22:36:15 Sailfish kernel: input: qpnp_pon as /devices/virtual/input/input0<br>
Feb 21 22:36:15 Sailfish kernel: pon_spare_reg: no parameters<br>
Feb 21 22:36:15 Sailfish kernel: qcom,qpnp-power-on qpnp-power-on-14: No PON config. specified<br>
Feb 21 22:36:15 Sailfish kernel: qcom,qpnp-power-on qpnp-power-on-14: PMIC@SID2 Power-on reason: Triggered from PON1 (secondary PMIC) and<br>
 'warm' boot<br>
Feb 21 22:36:15 Sailfish kernel: qcom,qpnp-power-on qpnp-power-on-14: PMIC@SID2: Power-off reason: Triggered from PS_HOLD (PS_HOLD/MSM co<br>
ntrolled shutdown)<br>
Feb 21 22:36:15 Sailfish kernel: PMIC@SID0: (null) v1.0 options: 2, 2, 0, 0<br>
Feb 21 22:36:15 Sailfish kernel: PMIC@SID2: PMI8950 v2.0 options: 0, 0, 0, 0<br>
Feb 21 22:36:15 Sailfish kernel: ipa ipa2_uc_state_check:296 uC interface not initialized<br>
Feb 21 22:36:15 Sailfish kernel: ipa ipa_sps_irq_control_all:938 EP (2) not allocated.<br>
Feb 21 22:36:15 Sailfish kernel: ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
Feb 21 22:36:15 Sailfish kernel: sps:BAM 0x0000000007904000 is registered.<br>
Feb 21 22:36:15 Sailfish kernel: sps:BAM 0x0000000007904000 (va:0xffffff8001840000) enabled: ver:0x27, number of pipes:20<br>
Feb 21 22:36:15 Sailfish kernel: IPA driver initialization was successful.<br>
Feb 21 22:36:15 Sailfish kernel: gdsc_venus: no parameters<br>
Feb 21 22:36:15 Sailfish kernel: gdsc_mdss: no parameters<br>
Feb 21 22:36:15 Sailfish kernel: gdsc_jpeg: no parameters<br>
Feb 21 22:36:15 Sailfish kernel: gdsc_vfe: no parameters<br>
Feb 21 22:36:15 Sailfish kernel: gdsc_vfe1: no parameters<br>
Feb 21 22:36:15 Sailfish kernel: gdsc_cpp: no parameters<br>
Feb 21 22:36:15 Sailfish kernel: gdsc_oxili_gx: no parameters<br>
Feb 21 22:36:15 Sailfish kernel: gdsc_oxili_gx: supplied by msm_gfx_ldo<br>
Feb 21 22:36:15 Sailfish kernel: gdsc_venus_core0: fast normal<br>
Feb 21 22:36:15 Sailfish kernel: gdsc_oxili_cx: no parameters<br>
Feb 21 22:36:15 Sailfish kernel: gdsc_usb30: no parameters<br>
Feb 21 22:36:15 Sailfish kernel: mdss_pll_probe: MDSS pll label = MDSS DSI 0 PLL<br>
Feb 21 22:36:15 Sailfish kernel: dsi_pll_clock_register_8996: Registered DSI PLL ndx=0 clocks successfully<br>
Feb 21 22:36:15 Sailfish kernel: mdss_pll_probe: MDSS pll label = MDSS DSI 1 PLL<br>
Feb 21 22:36:15 Sailfish kernel: pll_is_pll_locked_8996: DSI PLL ndx=1 status=0 failed to Lock<br>
Feb 21 22:36:15 Sailfish kernel: dsi_pll_clock_register_8996: Registered DSI PLL ndx=1 clocks successfully<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu 1e00000.qcom,iommu: device apps_iommu (model: 500) mapped at ffffff8001e00000, with 21 ctx ban<br>
ks<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e20000.qcom,iommu-ctx: context adsp_elf using bank 0<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e21000.qcom,iommu-ctx: context adsp_sec_pixel using bank 1<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e22000.qcom,iommu-ctx: context mdp_1 using bank 2<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e23000.qcom,iommu-ctx: context venus_fw using bank 3<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e24000.qcom,iommu-ctx: context venus_sec_non_pixel using bank 4<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e25000.qcom,iommu-ctx: context venus_sec_bitstream using bank 5<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e26000.qcom,iommu-ctx: context venus_sec_pixel using bank 6<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e28000.qcom,iommu-ctx: context pronto_pil using bank 8<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e29000.qcom,iommu-ctx: context q6 using bank 9<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e2a000.qcom,iommu-ctx: context periph_rpm using bank 10<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e2b000.qcom,iommu-ctx: context lpass using bank 11<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e2f000.qcom,iommu-ctx: context adsp_io using bank 15<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e30000.qcom,iommu-ctx: context adsp_opendsp using bank 16<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e31000.qcom,iommu-ctx: context adsp_shared using bank 17<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e32000.qcom,iommu-ctx: context cpp using bank 18<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e33000.qcom,iommu-ctx: context jpeg_enc0 using bank 19<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e34000.qcom,iommu-ctx: context vfe using bank 20<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e35000.qcom,iommu-ctx: context mdp_0 using bank 21<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e36000.qcom,iommu-ctx: context venus_ns using bank 22<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e38000.qcom,iommu-ctx: context ipa using bank 24<br>
Feb 21 22:36:15 Sailfish kernel: msm_iommu_ctx 1e37000.qcom,iommu-ctx: context access_control using bank 23<br>
Feb 21 22:36:15 Sailfish kernel: /soc/qcom,cam_smmu/msm_cam_smmu_cb1: could not get #iommu-cells for /soc/qcom,iommu@1e00000<br>
Feb 21 22:36:15 Sailfish kernel: /soc/qcom,cam_smmu/msm_cam_smmu_cb3: could not get #iommu-cells for /soc/qcom,iommu@1e00000<br>
Feb 21 22:36:15 Sailfish kernel: /soc/qcom,cam_smmu/msm_cam_smmu_cb4: could not get #iommu-cells for /soc/qcom,iommu@1e00000<br>
Feb 21 22:36:15 Sailfish kernel: adreno_idler: version 1.1 by arter97<br>
Feb 21 22:36:15 Sailfish kernel: Advanced Linux Sound Architecture Driver Initialized.<br>
Feb 21 22:36:15 Sailfish kernel: cfg80211: Calling CRDA to update world regulatory domain<br>
Feb 21 22:36:15 Sailfish kernel: cfg80211: World regulatory domain updated:<br>
Feb 21 22:36:15 Sailfish kernel: cfg80211:  DFS Master region: unset<br>
Feb 21 22:36:15 Sailfish kernel: cfg80211:   (start_freq - end_freq @ bandwidth), (max_antenna_gain, max_eirp), (dfs_cac_time)<br>
Feb 21 22:36:15 Sailfish kernel: cfg80211:   (2402000 KHz - 2472000 KHz @ 40000 KHz), (N/A, 2000 mBm), (N/A)<br>
Feb 21 22:36:15 Sailfish kernel: cfg80211:   (2457000 KHz - 2482000 KHz @ 40000 KHz), (N/A, 2000 mBm), (N/A)<br>
Feb 21 22:36:15 Sailfish kernel: cfg80211:   (2474000 KHz - 2494000 KHz @ 20000 KHz), (N/A, 2000 mBm), (N/A)<br>
Feb 21 22:36:15 Sailfish kernel: cfg80211:   (5170000 KHz - 5250000 KHz @ 80000 KHz), (N/A, 2000 mBm), (N/A)<br>
Feb 21 22:36:15 Sailfish kernel: cfg80211:   (5250000 KHz - 5330000 KHz @ 80000 KHz), (N/A, 2000 mBm), (N/A)<br>
Feb 21 22:36:15 Sailfish kernel: cfg80211:   (5490000 KHz - 5710000 KHz @ 80000 KHz), (N/A, 2000 mBm), (N/A)<br>
Feb 21 22:36:15 Sailfish kernel: cfg80211:   (5735000 KHz - 5835000 KHz @ 80000 KHz), (N/A, 2000 mBm), (N/A)<br>
Feb 21 22:36:15 Sailfish kernel: cfg80211:   (57240000 KHz - 63720000 KHz @ 2160000 KHz), (N/A, 0 mBm), (N/A)<br>
Feb 21 22:36:15 Sailfish kernel: pcie:pcie_init.<br>
Feb 21 22:36:15 Sailfish kernel: ibb_reg: 4600 -- 6000 mV at 5500 mV<br>
Feb 21 22:36:15 Sailfish kernel: lab_reg: 4600 -- 6000 mV at 5500 mV<br>
Feb 21 22:36:15 Sailfish kernel: Switched to clocksource arch_sys_counter<br>
Feb 21 22:36:15 Sailfish kernel: bcl_peripheral:bcl_perph_init BCL Initialized<br>
Feb 21 22:36:15 Sailfish kernel: NET: Registered protocol family 2<br>
Feb 21 22:36:15 Sailfish kernel: TCP established hash table entries: 32768 (order: 6, 262144 bytes)<br>
Feb 21 22:36:15 Sailfish kernel: TCP bind hash table entries: 32768 (order: 7, 524288 bytes)<br>
Feb 21 22:36:15 Sailfish kernel: TCP: Hash tables configured (established 32768 bind 32768)<br>
Feb 21 22:36:15 Sailfish kernel: TCP: reno registered<br>
Feb 21 22:36:15 Sailfish kernel: UDP hash table entries: 2048 (order: 4, 65536 bytes)<br>
Feb 21 22:36:15 Sailfish kernel: UDP-Lite hash table entries: 2048 (order: 4, 65536 bytes)<br>
Feb 21 22:36:15 Sailfish kernel: NET: Registered protocol family 1<br>
Feb 21 22:36:15 Sailfish kernel: PCI: CLS 0 bytes, default 64<br>
Feb 21 22:36:15 Sailfish kernel: gcc-mdss-8953 1800000.qcom,gcc-mdss: Registered GCC MDSS clocks.<br>
Feb 21 22:36:15 Sailfish kernel: Trying to unpack rootfs image as initramfs...<br>
Feb 21 22:36:15 Sailfish kernel: Freeing initrd memory: 820K<br>
Feb 21 22:36:15 Sailfish kernel: futex hash table entries: 2048 (order: 5, 131072 bytes)<br>
Feb 21 22:36:15 Sailfish kernel: Initialise system trusted keyring<br>
Feb 21 22:36:15 Sailfish kernel: audit: initializing netlink subsys (disabled)<br>
Feb 21 22:36:15 Sailfish kernel: audit: type=2000 audit(0.683:1): initialized<br>
Feb 21 22:36:15 Sailfish kernel: vmscan: error setting kswapd cpu affinity mask<br>
Feb 21 22:36:15 Sailfish kernel: zbud: loaded<br>
Feb 21 22:36:15 Sailfish kernel: VFS: Disk quotas dquot_6.5.2<br>
Feb 21 22:36:15 Sailfish kernel: Dquot-cache hash table entries: 512 (order 0, 4096 bytes)<br>
Feb 21 22:36:15 Sailfish kernel: squashfs: version 4.0 (2009/01/31) Phillip Lougher<br>
Feb 21 22:36:15 Sailfish kernel: exFAT: Version 1.2.9<br>
Feb 21 22:36:15 Sailfish kernel: fuse init (API version 7.23)<br>
Feb 21 22:36:15 Sailfish kernel: msgmni has been set to 5615<br>
Feb 21 22:36:15 Sailfish kernel: SELinux:  Registering netfilter hooks<br>
Feb 21 22:36:15 Sailfish kernel: Key type asymmetric registered<br>
Feb 21 22:36:15 Sailfish kernel: Asymmetric key parser 'x509' registered<br>
Feb 21 22:36:15 Sailfish kernel: Block layer SCSI generic (bsg) driver version 0.4 loaded (major 247)<br>
Feb 21 22:36:15 Sailfish kernel: io scheduler noop registered<br>
Feb 21 22:36:15 Sailfish kernel: io scheduler deadline registered<br>
Feb 21 22:36:15 Sailfish kernel: io scheduler cfq registered<br>
Feb 21 22:36:15 Sailfish kernel: io scheduler zen registered<br>
Feb 21 22:36:15 Sailfish kernel: io scheduler fiops registered<br>
Feb 21 22:36:15 Sailfish kernel: io scheduler sio registered<br>
Feb 21 22:36:15 Sailfish kernel: io scheduler maple registered (default)<br>
Feb 21 22:36:15 Sailfish kernel: io scheduler bfq registered<br>
Feb 21 22:36:15 Sailfish kernel: BFQ I/O-scheduler: v7r8<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b6000.i2c: msm_bus_scale_register_client(mstr-id:86):0x2 (ok)<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b6000.i2c: NACK: slave not responding, ensure its powered: msgs(n:2 cur:0 tx) bc(rx:1<br>
 tx:1) mode:FIFO slv_addr:0x39 MSTR_STS:0x0d1300c8 OPER:0x00000010<br>
Feb 21 22:36:15 Sailfish kernel: msm_dba_helper_i2c_read: i2c read failed<br>
Feb 21 22:36:15 Sailfish kernel: adv7533_read: read err: addr 0x39, reg 0x0, size 0x1<br>
Feb 21 22:36:15 Sailfish kernel: adv7533_probe: Failed to read chip rev<br>
Feb 21 22:36:15 Sailfish kernel: adv7533: probe of 2-0039 failed with error -5<br>
Feb 21 22:36:15 Sailfish kernel: msm_dss_get_res_byname: 'vbif_nrt_phys' resource not found<br>
Feb 21 22:36:15 Sailfish kernel: mdss_mdp_probe+0x228/0x1108-msm_dss_ioremap_byname: 'vbif_nrt_phys' msm_dss_get_res_byname failed<br>
<br>
Feb 21 22:36:15 Sailfish kernel: mdss_mdp_irq_clk_register: unable to get clk: lut_clk<br>
Feb 21 22:36:15 Sailfish kernel: No change in context(0==0), skip<br>
Feb 21 22:36:15 Sailfish kernel: mdss_mdp_pipe_addr_setup: type:0 ftchid:-1 xinid:0 num:0 rect:0 ndx:0x1 prio:0<br>
Feb 21 22:36:15 Sailfish kernel: mdss_mdp_pipe_addr_setup: type:1 ftchid:-1 xinid:1 num:3 rect:0 ndx:0x8 prio:1<br>
Feb 21 22:36:15 Sailfish kernel: mdss_mdp_pipe_addr_setup: type:1 ftchid:-1 xinid:5 num:4 rect:0 ndx:0x10 prio:2<br>
Feb 21 22:36:15 Sailfish kernel: mdss_mdp_pipe_addr_setup: type:2 ftchid:-1 xinid:2 num:6 rect:0 ndx:0x40 prio:3<br>
Feb 21 22:36:15 Sailfish kernel: mdss_mdp_pipe_addr_setup: type:3 ftchid:-1 xinid:7 num:10 rect:0 ndx:0x400 prio:0<br>
Feb 21 22:36:15 Sailfish kernel: mdss_mdp_parse_dt_handler: Error from prop qcom,mdss-pipe-sw-reset-off : u32 array read<br>
Feb 21 22:36:15 Sailfish kernel: mdss_mdp_parse_dt_handler: Error from prop qcom,mdss-ib-factor-overlap : u32 array read<br>
Feb 21 22:36:15 Sailfish kernel: xlog_status: enable:1, panic:1, dump:2<br>
Feb 21 22:36:15 Sailfish kernel: mdss_mdp_probe: mdss version = 0x10100000, bootloader display is on, num 1, intf_sel=0x00000100<br>
Feb 21 22:36:15 Sailfish kernel: mdss_smmu_util_parse_dt_clock: clocks are not defined<br>
Feb 21 22:36:15 Sailfish kernel: mdss_smmu_probe: iommu v2 domain[0] mapping and clk register successful!<br>
Feb 21 22:36:15 Sailfish kernel: mdss_smmu_util_parse_dt_clock: clocks are not defined<br>
Feb 21 22:36:15 Sailfish kernel: mdss_smmu_probe: iommu v2 domain[2] mapping and clk register successful!<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_ctrl_probe: DSI Ctrl name = MDSS DSI CTRL-0<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_find_panel_of_node: cmdline:0:qcom,mdss_dsi_nt35532_fhd_video:1:none:cfg:single_dsi panel_name:<br>
qcom,mdss_dsi_nt35532_fhd_video<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_panel_init: Panel Name = nt35532 fhd video mode dsi panel<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_panel_timing_from_dt: found new timing "qcom,mdss_dsi_nt35532_fhd_video" (ffffffc0ae395000)<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_parse_dcs_cmds: failed, key=qcom,mdss-dsi-post-panel-on-command<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_parse_dcs_cmds: failed, key=qcom,mdss-dsi-timing-switch-command<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_panel_get_dsc_cfg_np: cannot find dsc config node:<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_parse_panel_features: ulps feature disabled<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_parse_panel_features: ulps during suspend feature disabled<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_parse_dms_config: dynamic switch feature enabled: 0<br>
Feb 21 22:36:15 Sailfish kernel: No valid panel-status-check-mode string<br>
Feb 21 22:36:15 Sailfish kernel: 1a94000.qcom,mdss_dsi_ctrl0 supply vdd not found, using dummy regulator<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_parse_ctrl_params:3947 Unable to read qcom,display-id, data=0000000000000000,len=20<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_parse_gpio_params: bklt_en gpio not specified<br>
Feb 21 22:36:15 Sailfish kernel: msm_dss_get_res_byname: 'dsi_phy_regulator' resource not found<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_retrieve_ctrl_resources+0x178/0x1fc-msm_dss_ioremap_byname: 'dsi_phy_regulator' msm_dss_<br>
get_res_byname failed<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_retrieve_ctrl_resources: ctrl_base=ffffff8001808000 ctrl_size=400 phy_base=ffffff800180a400 phy<br>
_size=580<br>
Feb 21 22:36:15 Sailfish kernel: dsi_panel_device_register: Using default BTA for ESD check<br>
Feb 21 22:36:15 Sailfish kernel: dsi_panel_device_register: Continuous splash enabled<br>
Feb 21 22:36:15 Sailfish kernel: mdss_register_panel: adding framebuffer device 1a94000.qcom,mdss_dsi_ctrl0<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_ctrl_probe: Dsi Ctrl-0 initialized, DSI rev:0x10040002, PHY rev:0x2<br>
Feb 21 22:36:15 Sailfish kernel: mdss_dsi_status_init: DSI status check interval:2500<br>
Feb 21 22:36:15 Sailfish kernel: mdss_register_panel: adding framebuffer device soc:qcom,mdss_wb_panel<br>
Feb 21 22:36:15 Sailfish kernel: mdss_fb_probe: fb0: split_mode:0 left:0 right:0<br>
Feb 21 22:36:15 Sailfish kernel: mdss_fb_register: FrameBuffer[0] 1080x1920 registered successfully!<br>
Feb 21 22:36:15 Sailfish kernel: mdss_fb_probe: fb1: split_mode:0 left:0 right:0<br>
Feb 21 22:36:15 Sailfish kernel: mdss_fb_register: FrameBuffer[1] 640x640 registered successfully!<br>
Feb 21 22:36:15 Sailfish kernel: mdss_mdp_splash_parse_dt: splash mem child node is not present<br>
Feb 21 22:36:15 Sailfish kernel: mdss_mdp_kcal_store_fb0_ctl panel name nt35532 fhd video mode dsi panel<br>
Feb 21 22:36:15 Sailfish kernel: mdss_mdp_kcal_store_fb0_ctl panel found...<br>
Feb 21 22:36:15 Sailfish kernel: kcal_ctrl_init: registered<br>
Feb 21 22:36:15 Sailfish kernel: IPC_RTR: msm_ipc_router_smd_driver_register Already driver registered IPCRTR<br>
Feb 21 22:36:15 Sailfish kernel: IPC_RTR: msm_ipc_router_smd_driver_register Already driver registered IPCRTR<br>
Feb 21 22:36:15 Sailfish kernel: In memshare_probe, Memshare probe success<br>
Feb 21 22:36:15 Sailfish kernel: msm_rpm_log_probe: OK<br>
Feb 21 22:36:15 Sailfish kernel: subsys-pil-tz soc:qcom,kgsl-hyp: for a506_zap segments only will be dumped.<br>
Feb 21 22:36:15 Sailfish kernel: subsys-pil-tz 1de0000.qcom,venus: for venus segments only will be dumped.<br>
Feb 21 22:36:15 Sailfish kernel: msm_serial_hs module loaded<br>
Feb 21 22:36:15 Sailfish kernel: platform 1c40000.qcom,kgsl-iommu:gfx3d_secure: assigned reserved memory node secure_region@0<br>
Feb 21 22:36:15 Sailfish kernel: Boeffla WL blocker: driver version 1.1.0 started<br>
Feb 21 22:36:15 Sailfish kernel: brd: module loaded<br>
Feb 21 22:36:15 Sailfish kernel: loop: module loaded<br>
Feb 21 22:36:15 Sailfish kernel: zram: Added device: zram0<br>
Feb 21 22:36:15 Sailfish kernel: QSEECOM: qseecom_probe: qseecom.qsee_version = 0x1000000<br>
Feb 21 22:36:15 Sailfish kernel: QSEECOM: qseecom_retrieve_ce_data: Device does not support PFE<br>
Feb 21 22:36:15 Sailfish kernel: QSEECOM: qseecom_probe: qseecom clocks handled by other subsystem<br>
Feb 21 22:36:15 Sailfish kernel: QSEECOM: qseecom_probe: qsee reentrancy support phase is not defined, setting to default 0<br>
Feb 21 22:36:15 Sailfish kernel: QSEECOM: qseecom_probe: qseecom.whitelist_support = 0<br>
Feb 21 22:36:15 Sailfish kernel: i2c-core: driver [tabla-i2c-core] using legacy suspend method<br>
Feb 21 22:36:15 Sailfish kernel: i2c-core: driver [tabla-i2c-core] using legacy resume method<br>
Feb 21 22:36:15 Sailfish kernel: i2c-core: driver [wcd9xxx-i2c-core] using legacy suspend method<br>
Feb 21 22:36:15 Sailfish kernel: i2c-core: driver [wcd9xxx-i2c-core] using legacy resume method<br>
Feb 21 22:36:15 Sailfish kernel: i2c-core: driver [tasha-i2c-core] using legacy suspend method<br>
Feb 21 22:36:15 Sailfish kernel: i2c-core: driver [tasha-i2c-core] using legacy resume method<br>
Feb 21 22:36:15 Sailfish kernel: QCE50: __qce_get_device_tree_data: BAM Apps EE is not defined, setting to default 1<br>
Feb 21 22:36:15 Sailfish kernel: qce 720000.qcedev: Qualcomm Crypto 5.3.3 device found @0x720000<br>
Feb 21 22:36:15 Sailfish kernel: qce 720000.qcedev: CE device = 0x0<br>
                                 , IO base, CE = 0xffffff8001f40000<br>
                                 , Consumer (IN) PIPE 2,    Producer (OUT) PIPE 3<br>
                                 IO base BAM = 0x          (null)<br>
                                 BAM IRQ 193<br>
                                 Engines Availability = 0x2010853<br>
Feb 21 22:36:15 Sailfish kernel: sps:BAM 0x0000000000704000 is registered.<br>
Feb 21 22:36:15 Sailfish kernel: sps:BAM 0x0000000000704000 (va:0xffffff8002100000) enabled: ver:0x27, number of pipes:8<br>
Feb 21 22:36:15 Sailfish kernel: QCE50: qce_sps_init:  Qualcomm MSM CE-BAM at 0x0000000000704000 irq 193<br>
Feb 21 22:36:15 Sailfish kernel: QCE50: __qce_get_device_tree_data: BAM Apps EE is not defined, setting to default 1<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: Qualcomm Crypto 5.3.3 device found @0x720000<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: CE device = 0x0<br>
                                 , IO base, CE = 0xffffff8002140000<br>
                                 , Consumer (IN) PIPE 4,    Producer (OUT) PIPE 5<br>
                                 IO base BAM = 0x          (null)<br>
                                 BAM IRQ 193<br>
                                 Engines Availability = 0x2010853<br>
Feb 21 22:36:15 Sailfish kernel: QCE50: qce_sps_init:  Qualcomm MSM CE-BAM at 0x0000000000704000 irq 193<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-ecb-aes<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-cbc-aes<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-ctr-aes<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-ecb-des<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-cbc-des<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-ecb-3des<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-cbc-3des<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-xts-aes<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-sha1<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-sha256<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-aead-hmac-sha1-cbc-aes<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-aead-hmac-sha1-cbc-des<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-aead-hmac-sha1-cbc-3des<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-aead-hmac-sha256-cbc-aes<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-aead-hmac-sha256-cbc-des<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-aead-hmac-sha256-cbc-3des<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-hmac-sha1<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-hmac-sha256<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-aes-ccm<br>
Feb 21 22:36:15 Sailfish kernel: qcrypto 720000.qcrypto: qcrypto-rfc4309-aes-ccm<br>
Feb 21 22:36:15 Sailfish kernel: qcom_ice_get_device_tree_data: No vdd-hba-supply regulator, assuming not needed<br>
Feb 21 22:36:15 Sailfish kernel: ICE IRQ = 194<br>
Feb 21 22:36:15 Sailfish kernel: SCSI Media Changer driver v0.25<br>
Feb 21 22:36:15 Sailfish kernel: tun: Universal TUN/TAP device driver, 1.6<br>
Feb 21 22:36:15 Sailfish kernel: tun: (C) 1999-2004 Max Krasnyansky maxk@qualcomm.com<br>
Feb 21 22:36:15 Sailfish kernel: PPP generic driver version 2.4.2<br>
Feb 21 22:36:15 Sailfish kernel: PPP BSD Compression module registered<br>
Feb 21 22:36:15 Sailfish kernel: PPP Deflate Compression module registered<br>
Feb 21 22:36:15 Sailfish kernel: PPP MPPE Compression module registered<br>
Feb 21 22:36:15 Sailfish kernel: NET: Registered protocol family 24<br>
Feb 21 22:36:15 Sailfish kernel: wcnss_wlan probed in built-in mode<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver asix<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver ax88179_178a<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver cdc_ether<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver net1080<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver cdc_subset<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver zaurus<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver cdc_ncm<br>
Feb 21 22:36:15 Sailfish kernel: msm_sharedmem: sharedmem_register_qmi: qmi init successful<br>
Feb 21 22:36:15 Sailfish kernel: scm_call failed: func id 0x42000c16, ret: -1, syscall returns: 0x0, 0x0, 0x0<br>
Feb 21 22:36:15 Sailfish kernel: hyp_assign_table: Failed to assign memory protection, ret = -5<br>
Feb 21 22:36:15 Sailfish kernel: msm_sharedmem: setup_shared_ram_perms: hyp_assign_phys failed IPA=0x0160x00000000f4900000 size=157<br>
2864 err=-5<br>
Feb 21 22:36:15 Sailfish kernel: msm_sharedmem: msm_sharedmem_probe: Device created for client 'rmtfs'<br>
Feb 21 22:36:15 Sailfish kernel: msm-dwc3 7000000.ssusb: unable to get dbm device<br>
Feb 21 22:36:15 Sailfish kernel: ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver<br>
Feb 21 22:36:15 Sailfish kernel: ehci-pci: EHCI PCI platform driver<br>
Feb 21 22:36:15 Sailfish kernel: ehci-msm: Qualcomm On-Chip EHCI Host Controller<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver cdc_acm<br>
Feb 21 22:36:15 Sailfish kernel: cdc_acm: USB Abstract Control Model driver for USB modems and ISDN adapters<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver usb-storage<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver ums-alauda<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver ums-cypress<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver ums-datafab<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver ums-freecom<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver ums-isd200<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver ums-jumpshot<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver ums-karma<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver ums-sddr09<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver ums-sddr55<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver ums-usbat<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver usbserial<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver usb_ehset_test<br>
Feb 21 22:36:15 Sailfish kernel: gbridge_init: gbridge_init successs.<br>
Feb 21 22:36:15 Sailfish kernel: mousedev: PS/2 mouse device common for all mice<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver xpad<br>
Feb 21 22:36:15 Sailfish kernel: tony_test:[ft5435_ts_init]<br>
Feb 21 22:36:15 Sailfish kernel: ~~~~~ ft5435_ts_probe start<br>
Feb 21 22:36:15 Sailfish kernel: [ft5435_ts_probe]CONFIG_FB is defined<br>
Feb 21 22:36:15 Sailfish kernel: [ft5435_ts_probe]CONFIG_PM is defined<br>
Feb 21 22:36:15 Sailfish kernel: [ft5435_get_dt_vkey]000<br>
Feb 21 22:36:15 Sailfish kernel: [ft5435_get_dt_vkey]111<br>
Feb 21 22:36:15 Sailfish kernel: [ft5435_get_dt_vkey]222<br>
Feb 21 22:36:15 Sailfish kernel: [ft5435_get_dt_vkey]333<br>
Feb 21 22:36:15 Sailfish kernel: [FTS]keycode = 172, x= 500, y=2040<br>
Feb 21 22:36:15 Sailfish kernel: [FTS]keycode = 139, x= 200, y=2040<br>
Feb 21 22:36:15 Sailfish kernel: [FTS]keycode = 158, x= 800, y=2040<br>
Feb 21 22:36:15 Sailfish kernel: [ft5435_get_dt_vkey]5555<br>
Feb 21 22:36:15 Sailfish kernel: input: ft5435_ts as /devices/soc/78b7000.i2c/i2c-3/3-0038/input/input1<br>
Feb 21 22:36:15 Sailfish kernel: i2c-msm-v2 78b7000.i2c: msm_bus_scale_register_client(mstr-id:86):0xf (ok)<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_ts 3-0038: Device ID = 0x54<br>
Feb 21 22:36:15 Sailfish kernel: sps: BAM device 0x0000000007884000 is not registered yet.<br>
Feb 21 22:36:15 Sailfish kernel: sps:BAM 0x0000000007884000 is registered.<br>
Feb 21 22:36:15 Sailfish kernel: sps:BAM 0x0000000007884000 (va:0xffffff8001e60000) enabled: ver:0x19, number of pipes:12<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_ts 3-0038: report rate = 110Hz<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_ts 3-0038: Firmware version = 10.0.0<br>
Feb 21 22:36:15 Sailfish kernel: [Fu]fw_vendor_id=0x51<br>
Feb 21 22:36:15 Sailfish kernel: upgrade,fts_fw_vendor_id=0x51<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_fw_upgrade_by_array_data, suspended=0<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_ts 3-0038: Current firmware: 0x0a.0.0<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_ts 3-0038: New firmware: 0x0a.0.0<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_ts 3-0038: Exiting fw upgrade...<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_fw_upgrade_by_array_data done<br>
Feb 21 22:36:15 Sailfish kernel: ~~~~~ tp_glove_register enable!!!!!<br>
Feb 21 22:36:15 Sailfish kernel: [fts]ft5435_fw_LockDownInfo_get_from_boot, fw_vendor_id=0x51<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_fw_LockDownInfo_get_from_boot, FTS_UPGRADE_LOOP ok is  i = 0<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0x34, j=0<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0x35, j=1<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0x32, j=2<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0x01, j=3<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0xc6, j=4<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0x01, j=5<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0x34, j=6<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_fw_LockDownInfo_get_from_boot: REG VAL = 0x01, j=7<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_fw_LockDownInfo_get_from_boot: reset the tp<br>
Feb 21 22:36:15 Sailfish kernel: tpd_probe, ft5x46_ctpm_LockDownInfo_get_from_boot, tp_lockdown_info=34353201c6013401<br>
Feb 21 22:36:15 Sailfish kernel: ft5435_ts 3-0038: Create proc entry success!<br>
Feb 21 22:36:15 Sailfish kernel: ~~~~~ ft5435_ts_probe end<br>
Feb 21 22:36:15 Sailfish kernel: [ TSP ] ist30xx_init()<br>
Feb 21 22:36:15 Sailfish kernel: [ TSP ] ### IMAGIS probe(ver:3.0.0.0, addr:0x50) ###<br>
Feb 21 22:36:15 Sailfish kernel: [ TSP ] ##### Device tree #####<br>
Feb 21 22:36:15 Sailfish kernel: [ TSP ]  reset gpio: 64<br>
Feb 21 22:36:15 Sailfish kernel: [ TSP ]  irq gpio: 65<br>
Feb 21 22:36:15 Sailfish kernel: [ TSP ] ist30xx_request_gpio<br>
Feb 21 22:36:15 Sailfish kernel: [ TSP ] unable to request reset gpio: 64<br>
Feb 21 22:36:15 Sailfish kernel: [ TSP ] Error, ist30xx init driver<br>
Feb 21 22:36:15 Sailfish kernel: input: s2w_pwrkey as /devices/virtual/input/input2<br>
Feb 21 22:36:15 Sailfish kernel: [sweep2wake]: sweep2wake_init done<br>
Feb 21 22:36:15 Sailfish kernel: --------gf_init start.--------<br>
Feb 21 22:36:15 Sailfish kernel: msm8953-pinctrl 1000000.pinctrl: invalid group "gpio135" for function "blsp_spi6"<br>
Feb 21 22:36:15 Sailfish kernel: msm8953-pinctrl 1000000.pinctrl: invalid group "gpio136" for function "blsp_spi6"<br>
Feb 21 22:36:15 Sailfish kernel: msm8953-pinctrl 1000000.pinctrl: invalid group "gpio137" for function "blsp_spi6"<br>
Feb 21 22:36:15 Sailfish kernel: msm8953-pinctrl 1000000.pinctrl: invalid group "gpio138" for function "blsp_spi6"<br>
Feb 21 22:36:15 Sailfish kernel: --------gf_probe start.--------<br>
Feb 21 22:36:15 Sailfish kernel: input: gf3208 as /devices/virtual/input/input3<br>
Feb 21 22:36:15 Sailfish kernel: --------gf_probe end---OK.--------<br>
Feb 21 22:36:15 Sailfish kernel:  status = 0x0<br>
Feb 21 22:36:15 Sailfish kernel: --------gf_init end---OK.--------<br>
Feb 21 22:36:15 Sailfish kernel: fpc1020 soc:fpc1020: fpc1020_probe: ok<br>
Feb 21 22:36:15 Sailfish kernel: fpc1020_init OK<br>
Feb 21 22:36:15 Sailfish kernel: input: hbtp_vm as /devices/virtual/input/input4<br>
Feb 21 22:36:15 Sailfish kernel: qcom,qpnp-rtc qpnp-rtc-8: rtc core: registered qpnp_rtc as rtc0<br>
Feb 21 22:36:15 Sailfish kernel: i2c /dev entries driver<br>
Feb 21 22:36:15 Sailfish kernel: lirc_dev: IR Remote Control driver registered, major 232<br>
Feb 21 22:36:15 Sailfish kernel: IR NEC protocol handler initialized<br>
Feb 21 22:36:15 Sailfish kernel: IR RC5(x/sz) protocol handler initialized<br>
Feb 21 22:36:15 Sailfish kernel: IR RC6 protocol handler initialized<br>
Feb 21 22:36:15 Sailfish kernel: IR JVC protocol handler initialized<br>
Feb 21 22:36:15 Sailfish kernel: IR Sony protocol handler initialized<br>
Feb 21 22:36:15 Sailfish kernel: IR SANYO protocol handler initialized<br>
Feb 21 22:36:15 Sailfish kernel: IR Sharp protocol handler initialized<br>
Feb 21 22:36:15 Sailfish kernel: IR LIRC bridge handler initialized<br>
Feb 21 22:36:15 Sailfish kernel: IR XMP protocol handler initialized<br>
Feb 21 22:36:15 Sailfish kernel: /soc/qcom,cam_smmu/msm_cam_smmu_cb1: could not get #iommu-cells for /soc/qcom,iommu@1e00000<br>
Feb 21 22:36:15 Sailfish kernel: /soc/qcom,cam_smmu/msm_cam_smmu_cb3: could not get #iommu-cells for /soc/qcom,iommu@1e00000<br>
Feb 21 22:36:15 Sailfish kernel: /soc/qcom,cam_smmu/msm_cam_smmu_cb4: could not get #iommu-cells for /soc/qcom,iommu@1e00000<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_get_dt_vreg_data:1115 number of entries is 0 or not present in dts<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_get_dt_vreg_data:1115 number of entries is 0 or not present in dts<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_get_dt_vreg_data:1115 number of entries is 0 or not present in dts<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_get_dt_vreg_data:1115 number of entries is 0 or not present in dts<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_get_dt_vreg_data:1115 number of entries is 0 or not present in dts<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_get_dt_vreg_data:1115 number of entries is 0 or not present in dts<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_get_dt_vreg_data:1115 number of entries is 0 or not present in dts<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_pinctrl_init:1277 Getting pinctrl handle failed<br>
Feb 21 22:36:15 Sailfish kernel: msm_actuator_platform_probe:1976 ERR:msm_actuator_platform_probe: Error in reading actuator pinctr<br>
l<br>
Feb 21 22:36:15 Sailfish kernel: qcom,actuator: probe of 1b0c000.qcom,cci:qcom,actuator@0 failed with error -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_config_single_vreg : can't find sub reg name<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_init:1426: hw_version = 0x10020005<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 158<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_irq:1778 MASTER_0 error 0x10000000<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_i2c_read:955 read_words = 0, exp words = 1<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_i2c_read_bytes:1038 failed rc -22<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 173 error<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe read_eeprom_memory failed<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_config_single_vreg : can't find sub reg name<br>
Feb 21 22:36:15 Sailfish kernel: qcom,eeprom: probe of 1b0c000.qcom,cci:qcom,eeprom@0 failed with error -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_config_single_vreg : can't find sub reg name<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_init:1426: hw_version = 0x10020005<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 158<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_config_single_vreg : can't find sub reg name<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_config_single_vreg : can't find sub reg name<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_init:1426: hw_version = 0x10020005<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 158<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_config_single_vreg : can't find sub reg name<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_init:1426: hw_version = 0x10020005<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 158<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_irq:1784 MASTER_1 error 0x40000000<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_i2c_read:955 read_words = 0, exp words = 1<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_i2c_read_bytes:1038 failed rc -22<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 173 error<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe read_eeprom_memory failed<br>
Feb 21 22:36:15 Sailfish kernel: qcom,eeprom: probe of 1b0c000.qcom,cci:qcom,eeprom@3 failed with error -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_init:1426: hw_version = 0x10020005<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 158<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_irq:1784 MASTER_1 error 0x40000000<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_i2c_read:955 read_words = 0, exp words = 1<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_i2c_read_bytes:1038 failed rc -22<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 173 error<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe read_eeprom_memory failed<br>
Feb 21 22:36:15 Sailfish kernel: qcom,eeprom: probe of 1b0c000.qcom,cci:qcom,eeprom@4 failed with error -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_init:1426: hw_version = 0x10020005<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 158<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_config_single_vreg : can't find sub reg name<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_init:1426: hw_version = 0x10020005<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 158<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_config_single_vreg : can't find sub reg name<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_config_single_vreg : can't find sub reg name<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_init:1426: hw_version = 0x10020005<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 158<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_irq:1778 MASTER_0 error 0x10000000<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_i2c_read:955 read_words = 0, exp words = 1<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_i2c_read_bytes:1038 failed rc -22<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 173 error<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe read_eeprom_memory failed<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_config_single_vreg : can't find sub reg name<br>
Feb 21 22:36:15 Sailfish kernel: qcom,eeprom: probe of 1b0c000.qcom,cci:qcom,eeprom@7 failed with error -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_config_single_vreg : can't find sub reg name<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_init:1426: hw_version = 0x10020005<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 158<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: qcom,slave-addr = 0xB0<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_config_single_vreg : can't find sub reg name<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe qcom,i2c-freq-mode read fail. Setting to 0 -22<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_config_single_vreg : can't find sub reg name<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_init:1426: hw_version = 0x10020005<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 158<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_irq:1778 MASTER_0 error 0x10000000<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_i2c_read:955 read_words = 0, exp words = 1<br>
Feb 21 22:36:15 Sailfish kernel: msm_cci_i2c_read_bytes:1038 failed rc -22<br>
Feb 21 22:36:15 Sailfish kernel: read_eeprom_memory 173 error<br>
Feb 21 22:36:15 Sailfish kernel: msm_eeprom_platform_probe read_eeprom_memory failed<br>
Feb 21 22:36:15 Sailfish kernel: msm_camera_config_single_vreg : can't find sub reg name<br>
Feb 21 22:36:15 Sailfish kernel: qcom,eeprom: probe of 1b0c000.qcom,cci:qcom,eeprom@9 failed with error -22<br>
Feb 21 22:36:15 Sailfish kernel: MSM-CPP cpp_init_hardware:869 CPP HW Version: 0x40030003<br>
Feb 21 22:36:15 Sailfish kernel: MSM-CPP cpp_init_hardware:887 stream_cnt:0<br>
Feb 21 22:36:15 Sailfish kernel: __msm_jpeg_init:1537] Jpeg Device id 0<br>
Feb 21 22:36:15 Sailfish kernel: FG: fg_probe: FG Probe success - FG Revision DIG:3.1 ANA:1.2 PMIC subtype=17<br>
Feb 21 22:36:15 Sailfish kernel: thermal thermal_zone1: failed to read out thermal zone 1<br>
Feb 21 22:36:15 Sailfish kernel: thermal thermal_zone2: failed to read out thermal zone 2<br>
Feb 21 22:36:15 Sailfish kernel: thermal thermal_zone3: failed to read out thermal zone 3<br>
Feb 21 22:36:15 Sailfish kernel: qpnp_vadc_read: no vadc_chg_vote found<br>
Feb 21 22:36:15 Sailfish kernel: qpnp_vadc_get_temp: VADC read error with -22<br>
Feb 21 22:36:15 Sailfish kernel: thermal thermal_zone4: failed to read out thermal zone 4<br>
Feb 21 22:36:15 Sailfish kernel: device-mapper: uevent: version 1.0.3<br>
Feb 21 22:36:15 Sailfish kernel: device-mapper: ioctl: 4.28.0-ioctl (2014-09-17) initialised: dm-devel@redhat.com<br>
Feb 21 22:36:15 Sailfish kernel: device-mapper: req-crypt: dm-req-crypt successfully initalized.<br>
Feb 21 22:36:15 Sailfish kernel: ------------[ cut here ]------------<br>
Feb 21 22:36:15 Sailfish kernel: WARNING: CPU: 7 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi<br>
/msm8953/fs/sysfs/dir.c:31 sysfs_warn_dup+0x70/0x88()<br>
Feb 21 22:36:15 Sailfish kernel: sysfs: cannot create duplicate filename '/devices/system/cpu/cpu0/cpufreq/stats'<br>
Feb 21 22:36:15 Sailfish kernel: Modules linked in:<br>
Feb 21 22:36:15 Sailfish kernel: CPU: 7 PID: 1 Comm: swapper/0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
Feb 21 22:36:15 Sailfish kernel: Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
Feb 21 22:36:15 Sailfish kernel: Call trace:<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a2c4] show_stack+0x20/0x28<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000256774] sysfs_warn_dup+0x70/0x88<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000256f10] internal_create_group+0xc8/0x1f4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000257068] sysfs_create_group+0x2c/0x38<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0009b3ac0] __cpufreq_stats_create_table.part.3+0x78/0x1ac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc001544404] cpufreq_stats_init+0x180/0x2bc<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e03928] kernel_init+0x20/0xe0<br>
Feb 21 22:36:15 Sailfish kernel: ---[ end trace baf5d4897624fa09 ]---<br>
Feb 21 22:36:15 Sailfish kernel: ------------[ cut here ]------------<br>
Feb 21 22:36:15 Sailfish kernel: WARNING: CPU: 7 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi<br>
/msm8953/fs/sysfs/dir.c:31 sysfs_warn_dup+0x70/0x88()<br>
Feb 21 22:36:15 Sailfish kernel: sysfs: cannot create duplicate filename '/devices/system/cpu/cpu0/cpufreq/stats'<br>
Feb 21 22:36:15 Sailfish kernel: Modules linked in:<br>
Feb 21 22:36:15 Sailfish kernel: CPU: 7 PID: 1 Comm: swapper/0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
Feb 21 22:36:15 Sailfish kernel: Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
Feb 21 22:36:15 Sailfish kernel: Call trace:<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a2c4] show_stack+0x20/0x28<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000256774] sysfs_warn_dup+0x70/0x88<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000256f10] internal_create_group+0xc8/0x1f4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000257068] sysfs_create_group+0x2c/0x38<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0009b3ac0] __cpufreq_stats_create_table.part.3+0x78/0x1ac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc001544404] cpufreq_stats_init+0x180/0x2bc<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e03928] kernel_init+0x20/0xe0<br>
Feb 21 22:36:15 Sailfish kernel: ---[ end trace baf5d4897624fa0a ]---<br>
Feb 21 22:36:15 Sailfish kernel: ------------[ cut here ]------------<br>
Feb 21 22:36:15 Sailfish kernel: WARNING: CPU: 7 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi<br>
/msm8953/fs/sysfs/dir.c:31 sysfs_warn_dup+0x70/0x88()<br>
Feb 21 22:36:15 Sailfish kernel: sysfs: cannot create duplicate filename '/devices/system/cpu/cpu0/cpufreq/stats'<br>
Feb 21 22:36:15 Sailfish kernel: Modules linked in:<br>
Feb 21 22:36:15 Sailfish kernel: CPU: 7 PID: 1 Comm: swapper/0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
Feb 21 22:36:15 Sailfish kernel: Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
Feb 21 22:36:15 Sailfish kernel: Call trace:<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a2c4] show_stack+0x20/0x28<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000256774] sysfs_warn_dup+0x70/0x88<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000256f10] internal_create_group+0xc8/0x1f4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000257068] sysfs_create_group+0x2c/0x38<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0009b3ac0] __cpufreq_stats_create_table.part.3+0x78/0x1ac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc001544404] cpufreq_stats_init+0x180/0x2bc<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e03928] kernel_init+0x20/0xe0<br>
Feb 21 22:36:15 Sailfish kernel: ---[ end trace baf5d4897624fa0b ]---<br>
Feb 21 22:36:15 Sailfish kernel: ------------[ cut here ]------------<br>
Feb 21 22:36:15 Sailfish kernel: WARNING: CPU: 7 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi<br>
/msm8953/fs/sysfs/dir.c:31 sysfs_warn_dup+0x70/0x88()<br>
Feb 21 22:36:15 Sailfish kernel: sysfs: cannot create duplicate filename '/devices/system/cpu/cpu0/cpufreq/stats'<br>
Feb 21 22:36:15 Sailfish kernel: Modules linked in:<br>
Feb 21 22:36:15 Sailfish kernel: CPU: 7 PID: 1 Comm: swapper/0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
Feb 21 22:36:15 Sailfish kernel: Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
Feb 21 22:36:15 Sailfish kernel: Call trace:<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a2c4] show_stack+0x20/0x28<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000256774] sysfs_warn_dup+0x70/0x88<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000256f10] internal_create_group+0xc8/0x1f4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000257068] sysfs_create_group+0x2c/0x38<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0009b3ac0] __cpufreq_stats_create_table.part.3+0x78/0x1ac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc001544404] cpufreq_stats_init+0x180/0x2bc<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e03928] kernel_init+0x20/0xe0<br>
Feb 21 22:36:15 Sailfish kernel: ---[ end trace baf5d4897624fa0c ]---<br>
Feb 21 22:36:15 Sailfish kernel: ------------[ cut here ]------------<br>
Feb 21 22:36:15 Sailfish kernel: WARNING: CPU: 7 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi<br>
/msm8953/fs/sysfs/dir.c:31 sysfs_warn_dup+0x70/0x88()<br>
Feb 21 22:36:15 Sailfish kernel: sysfs: cannot create duplicate filename '/devices/system/cpu/cpu0/cpufreq/stats'<br>
Feb 21 22:36:15 Sailfish kernel: Modules linked in:<br>
Feb 21 22:36:15 Sailfish kernel: CPU: 7 PID: 1 Comm: swapper/0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
Feb 21 22:36:15 Sailfish kernel: Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
Feb 21 22:36:15 Sailfish kernel: Call trace:<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a2c4] show_stack+0x20/0x28<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000256774] sysfs_warn_dup+0x70/0x88<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000256f10] internal_create_group+0xc8/0x1f4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000257068] sysfs_create_group+0x2c/0x38<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0009b3ac0] __cpufreq_stats_create_table.part.3+0x78/0x1ac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc001544404] cpufreq_stats_init+0x180/0x2bc<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e03928] kernel_init+0x20/0xe0<br>
Feb 21 22:36:15 Sailfish kernel: ---[ end trace baf5d4897624fa0d ]---<br>
Feb 21 22:36:15 Sailfish kernel: ------------[ cut here ]------------<br>
Feb 21 22:36:15 Sailfish kernel: WARNING: CPU: 7 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi<br>
/msm8953/fs/sysfs/dir.c:31 sysfs_warn_dup+0x70/0x88()<br>
Feb 21 22:36:15 Sailfish kernel: sysfs: cannot create duplicate filename '/devices/system/cpu/cpu0/cpufreq/stats'<br>
Feb 21 22:36:15 Sailfish kernel: Modules linked in:<br>
Feb 21 22:36:15 Sailfish kernel: CPU: 7 PID: 1 Comm: swapper/0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
Feb 21 22:36:15 Sailfish kernel: Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
Feb 21 22:36:15 Sailfish kernel: Call trace:<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a2c4] show_stack+0x20/0x28<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000256774] sysfs_warn_dup+0x70/0x88<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000256f10] internal_create_group+0xc8/0x1f4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000257068] sysfs_create_group+0x2c/0x38<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0009b3ac0] __cpufreq_stats_create_table.part.3+0x78/0x1ac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc001544404] cpufreq_stats_init+0x180/0x2bc<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e03928] kernel_init+0x20/0xe0<br>
Feb 21 22:36:15 Sailfish kernel: ---[ end trace baf5d4897624fa0e ]---<br>
Feb 21 22:36:15 Sailfish kernel: ------------[ cut here ]------------<br>
Feb 21 22:36:15 Sailfish kernel: WARNING: CPU: 7 PID: 1 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaomi<br>
/msm8953/fs/sysfs/dir.c:31 sysfs_warn_dup+0x70/0x88()<br>
Feb 21 22:36:15 Sailfish kernel: sysfs: cannot create duplicate filename '/devices/system/cpu/cpu0/cpufreq/stats'<br>
Feb 21 22:36:15 Sailfish kernel: Modules linked in:<br>
Feb 21 22:36:15 Sailfish kernel: CPU: 7 PID: 1 Comm: swapper/0 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
Feb 21 22:36:15 Sailfish kernel: Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
Feb 21 22:36:15 Sailfish kernel: Call trace:<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc00008a2c4] show_stack+0x20/0x28<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000256774] sysfs_warn_dup+0x70/0x88<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000256f10] internal_create_group+0xc8/0x1f4<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000257068] sysfs_create_group+0x2c/0x38<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0009b3ac0] __cpufreq_stats_create_table.part.3+0x78/0x1ac<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc001544404] cpufreq_stats_init+0x180/0x2bc<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000082c10] do_one_initcall+0x194/0x1b0<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc0014fcb98] kernel_init_freeable+0x1d0/0x284<br>
Feb 21 22:36:15 Sailfish kernel: [ffffffc000e03928] kernel_init+0x20/0xe0<br>
Feb 21 22:36:15 Sailfish kernel: ---[ end trace baf5d4897624fa0f ]---<br>
Feb 21 22:36:15 Sailfish kernel: sdhci: Secure Digital Host Controller Interface driver<br>
Feb 21 22:36:15 Sailfish kernel: sdhci: Copyright(c) Pierre Ossman<br>
Feb 21 22:36:15 Sailfish kernel: sdhci-pltfm: SDHCI platform and OF driver helper<br>
Feb 21 22:36:15 Sailfish kernel: qcom_ice_get_pdevice: found ice device ffffffc0ae094200<br>
Feb 21 22:36:15 Sailfish kernel: qcom_ice_get_pdevice: matching platform device ffffffc0af3ed800<br>
Feb 21 22:36:15 Sailfish kernel: qcom_ice 7803000.sdcc1ice: QC ICE 2.1.44 device found @0xffffff8001d70000<br>
Feb 21 22:36:15 Sailfish kernel: sdhci_msm 7824900.sdhci: No vmmc regulator found<br>
Feb 21 22:36:15 Sailfish kernel: sdhci_msm 7824900.sdhci: No vqmmc regulator found<br>
Feb 21 22:36:15 Sailfish kernel: mmc0: SDHCI controller on 7824900.sdhci [7824900.sdhci] using 64-bit ADMA in CMDQ mode<br>
Feb 21 22:36:15 Sailfish kernel: sdhci_msm 7864900.sdhci: sdhci_msm_probe: ICE device is not enabled<br>
Feb 21 22:36:15 Sailfish kernel: sdhci_msm 7864900.sdhci: No vmmc regulator found<br>
Feb 21 22:36:15 Sailfish kernel: sdhci_msm 7864900.sdhci: No vqmmc regulator found<br>
Feb 21 22:36:15 Sailfish kernel: mmc1: SDHCI controller on 7864900.sdhci [7864900.sdhci] using 64-bit ADMA in legacy mode<br>
Feb 21 22:36:15 Sailfish kernel: mmc0: Out-of-interrupt timeout is 50[ms]<br>
Feb 21 22:36:15 Sailfish kernel: mmc0: BKOPS_EN equals 0x2<br>
Feb 21 22:36:15 Sailfish kernel: mmc0: eMMC FW version: 0x07<br>
Feb 21 22:36:15 Sailfish kernel: mmc0: CMDQ supported: depth: 16<br>
Feb 21 22:36:15 Sailfish kernel: mmc0: cache barrier support 0 flush policy 0<br>
Feb 21 22:36:15 Sailfish kernel: cmdq_host_alloc_tdl: desc_size: 768 data_sz: 253952 slot-sz: 24<br>
Feb 21 22:36:15 Sailfish kernel: mmc0: CMDQ enabled on card<br>
Feb 21 22:36:15 Sailfish kernel: mmc0: new HS400 MMC card at address 0001<br>
Feb 21 22:36:15 Sailfish kernel: sdhci_msm_pm_qos_cpu_init (): voted for group #0 (mask=0xf) latency=2<br>
Feb 21 22:36:15 Sailfish kernel: sdhci_msm_pm_qos_cpu_init (): voted for group #1 (mask=0xf0) latency=2<br>
Feb 21 22:36:15 Sailfish kernel: mmcblk0: mmc0:0001 RX1BMB 29.1 GiB<br>
Feb 21 22:36:15 Sailfish kernel: mmcblk0rpmb: mmc0:0001 RX1BMB partition 3 4.00 MiB<br>
Feb 21 22:36:15 Sailfish kernel:  mmcblk0: p1 p2 p3 p4 p5 p6 p7 p8 p9 p10 p11 p12 p13 p14 p15 p16 p17 p18 p19 p20 p21 p22 p23 p24 p25 p26<br>
 p27 p28 p29 p30 p31 p32 p33 p34 p35 p36 p37 p38 p39 p40 p41 p42 p43 p44 p45 p46 p47 p48 p49<br>
Feb 21 22:36:15 Sailfish kernel: qcom,leds-qpnp: probe of leds-qpnp-21 failed with error -10<br>
Feb 21 22:36:15 Sailfish kernel: tz_log 8600720.tz-log: Hyp log service is not supported<br>
Feb 21 22:36:15 Sailfish kernel: hidraw: raw HID events driver (C) Jiri Kosina<br>
Feb 21 22:36:15 Sailfish kernel: usbcore: registered new interface driver usbhid<br>
Feb 21 22:36:15 Sailfish kernel: usbhid: USB HID core driver<br>
Feb 21 22:36:15 Sailfish kernel: ashmem: initialized<br>
Feb 21 22:36:15 Sailfish kernel: logger: created 256K log 'log_main'<br>
Feb 21 22:36:15 Sailfish kernel: logger: created 256K log 'log_events'<br>
Feb 21 22:36:15 Sailfish kernel: logger: created 256K log 'log_radio'<br>
Feb 21 22:36:15 Sailfish kernel: logger: created 256K log 'log_system'<br>
Feb 21 22:36:15 Sailfish kernel: qpnp_coincell_charger_show_state: enabled=Y, voltage=3200 mV, resistance=2100 ohm<br>
Feb 21 22:36:15 Sailfish kernel: bimc-bwmon 408000.qcom,cpu-bwmon: BW HWmon governor registered.<br>
Feb 21 22:36:15 Sailfish kernel: devfreq soc:qcom,cpubw: Couldn't update frequency transition information.<br>
Feb 21 22:36:15 Sailfish kernel: devfreq soc:qcom,mincpubw: Couldn't update frequency transition information.<br>
Feb 21 22:36:15 Sailfish kernel: coresight-fuse a601c.fuse: Fuse initialized<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6010000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6011000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6012000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6013000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6014000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6015000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6016000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6017000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6018000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6019000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 601a000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 601b000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 601c000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 601d000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 601e000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 601f000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6198000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6199000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 619a000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 619b000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 61b8000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 61b9000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 61ba000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 61bb000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6128000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6124000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6134000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 6139000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 613c000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-cti: probe of 610c000.cti failed with error -1<br>
Feb 21 22:36:15 Sailfish kernel: coresight-csr 6001000.csr: CSR initialized<br>
Feb 21 22:36:16 Sailfish kernel: coresight-tmc: probe of 6028000.tmc failed with error -1<br>
Feb 21 22:36:16 Sailfish kernel: coresight-tmc 6027000.tmc: failed to get flush cti<br>
Feb 21 22:36:16 Sailfish kernel: coresight-tmc 6027000.tmc: failed to get reset cti<br>
Feb 21 22:36:16 Sailfish kernel: coresight-tmc 6027000.tmc: TMC initialized<br>
Feb 21 22:36:16 Sailfish kernel: nidnt boot config: 2<br>
Feb 21 22:36:16 Sailfish kernel: NIDnT disabled, only sd mode supported.<br>
Feb 21 22:36:16 Sailfish kernel: coresight-tpiu 6020000.tpiu: NIDnT hw support disabled<br>
Feb 21 22:36:16 Sailfish kernel: coresight-tpiu 6020000.tpiu: NIDnT on SDCARD only mode<br>
Feb 21 22:36:16 Sailfish kernel: coresight-tpiu 6020000.tpiu: TPIU initialized<br>
Feb 21 22:36:16 Sailfish kernel: coresight-replicator 6026000.replicator: REPLICATOR initialized<br>
Feb 21 22:36:16 Sailfish kernel: coresight-stm: probe of 6002000.stm failed with error -1<br>
Feb 21 22:36:16 Sailfish kernel: coresight-hwevent 6101000.hwevent: Hardware Event driver initialized<br>
Feb 21 22:36:16 Sailfish kernel: usbcore: registered new interface driver snd-usb-audio<br>
Feb 21 22:36:16 Sailfish kernel: msm-pcm-lpa soc:qcom,msm-pcm-lpa: msm_pcm_probe: dev name soc:qcom,msm-pcm-lpa<br>
Feb 21 22:36:16 Sailfish kernel: u32 classifier<br>
Feb 21 22:36:16 Sailfish kernel:     Actions configured<br>
Feb 21 22:36:16 Sailfish kernel: Netfilter messages via NETLINK v0.30.<br>
Feb 21 22:36:16 Sailfish kernel: nf_conntrack version 0.5.0 (16384 buckets, 65536 max)<br>
Feb 21 22:36:16 Sailfish kernel: ctnetlink v0.93: registering with nfnetlink.<br>
Feb 21 22:36:16 Sailfish kernel: xt_time: kernel timezone is -0000<br>
Feb 21 22:36:16 Sailfish kernel: wireguard: WireGuard 0.0.20180809 loaded. See www.wireguard.com for information.<br>
Feb 21 22:36:16 Sailfish kernel: wireguard: Copyright (C) 2015-2018 Jason A. Donenfeld Jason@zx2c4.com. All Rights Reserved.<br>
Feb 21 22:36:16 Sailfish kernel: ip_tables: (C) 2000-2006 Netfilter Core Team<br>
Feb 21 22:36:16 Sailfish kernel: arp_tables: (C) 2002 David S. Miller<br>
Feb 21 22:36:16 Sailfish kernel: TCP: cubic registered<br>
Feb 21 22:36:16 Sailfish kernel: TCP: highspeed registered<br>
Feb 21 22:36:16 Sailfish kernel: TCP: htcp registered<br>
Feb 21 22:36:16 Sailfish kernel: TCP: vegas registered<br>
Feb 21 22:36:16 Sailfish kernel: TCP: veno registered<br>
Feb 21 22:36:16 Sailfish kernel: TCP: scalable registered<br>
Feb 21 22:36:16 Sailfish kernel: TCP: lp registered<br>
Feb 21 22:36:16 Sailfish kernel: TCP: yeah registered<br>
Feb 21 22:36:16 Sailfish kernel: TCP: illinois registered<br>
Feb 21 22:36:16 Sailfish kernel: Initializing XFRM netlink socket<br>
Feb 21 22:36:16 Sailfish kernel: NET: Registered protocol family 10<br>
Feb 21 22:36:16 Sailfish kernel: mip6: Mobile IPv6<br>
Feb 21 22:36:16 Sailfish kernel: ip6_tables: (C) 2000-2006 Netfilter Core Team<br>
Feb 21 22:36:16 Sailfish kernel: sit: IPv6 over IPv4 tunneling driver<br>
Feb 21 22:36:16 Sailfish kernel: NET: Registered protocol family 17<br>
Feb 21 22:36:16 Sailfish kernel: NET: Registered protocol family 15<br>
Feb 21 22:36:16 Sailfish kernel: bridge: automatic filtering via arp/ip/ip6tables has been deprecated. Update your scripts to load br_net<br>
filter if you need this.<br>
Feb 21 22:36:16 Sailfish kernel: Bridge firewalling registered<br>
Feb 21 22:36:16 Sailfish kernel: Ebtables v2.0 registered<br>
Feb 21 22:36:16 Sailfish kernel: l2tp_core: L2TP core driver, V2.0<br>
Feb 21 22:36:16 Sailfish kernel: l2tp_ppp: PPPoL2TP kernel driver, V2.0<br>
Feb 21 22:36:16 Sailfish kernel: l2tp_ip: L2TP IP encapsulation support (L2TPv3)<br>
Feb 21 22:36:16 Sailfish kernel: l2tp_netlink: L2TP netlink interface<br>
Feb 21 22:36:16 Sailfish kernel: l2tp_eth: L2TP ethernet pseudowire support (L2TPv3)<br>
Feb 21 22:36:16 Sailfish kernel: l2tp_debugfs: L2TP debugfs support<br>
Feb 21 22:36:16 Sailfish kernel: l2tp_ip6: L2TP IP encapsulation support for IPv6 (L2TPv3)<br>
Feb 21 22:36:16 Sailfish kernel: 8021q: 802.1Q VLAN Support v1.8<br>
Feb 21 22:36:16 Sailfish kernel: NET: Registered protocol family 27<br>
Feb 21 22:36:16 Sailfish kernel: subsys-pil-tz a21b000.qcom,pronto: for wcnss segments only will be dumped.<br>
Feb 21 22:36:16 Sailfish kernel: pil-q6v5-mss 4080000.qcom,mss: for modem segments only will be dumped.<br>
Feb 21 22:36:16 Sailfish kernel: sps:BAM 0x0000000007104000 is registered.<br>
Feb 21 22:36:16 Sailfish kernel: qpnp-smbcharger qpnp-smbcharger-18: node /soc/qcom,spmi@200f000/qcom,pmi8950@2/qcom,qpnp-smbcharge<br>
r IO resource absent!<br>
Feb 21 22:36:16 Sailfish kernel: smbcharger_charger_otg: no parameters<br>
Feb 21 22:36:16 Sailfish kernel: SMBCHG: smbchg_hvdcp_enable_cb: smbchg_hvdcp_enable_cb  enable 0 last_enable 0<br>
Feb 21 22:36:16 Sailfish kernel: smbchg_hw_init:read OTG_CFG=78<br>
Feb 21 22:36:16 Sailfish kernel: SMBCHG: wait_for_usbin_uv: usbin uv didnt go to a lowered state, still at high, tries = 2, rc = 0<br>
<br>
Feb 21 22:36:16 Sailfish kernel: SMBCHG: rerun_apsd: wait for usbin uv failed rc = -22<br>
Feb 21 22:36:16 Sailfish kernel: ------------[ cut here ]------------<br>
Feb 21 22:36:16 Sailfish kernel: WARNING: CPU: 0 PID: 79 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaom<br>
i/msm8953/drivers/usb/dwc3/dwc3-msm.c:3590 dwc3_otg_sm_work+0x268/0x610()<br>
Feb 21 22:36:16 Sailfish kernel: Modules linked in:<br>
Feb 21 22:36:16 Sailfish kernel: CPU: 0 PID: 79 Comm: kworker/0:1 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
Feb 21 22:36:16 Sailfish kernel: Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
Feb 21 22:36:16 Sailfish kernel: Workqueue: events dwc3_otg_sm_work<br>
Feb 21 22:36:16 Sailfish kernel: Call trace:<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc00008a2c4] show_stack+0x20/0x28<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc0000a69ac] warn_slowpath_null+0x38/0x44<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc000774b58] dwc3_otg_sm_work+0x268/0x610<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc0000bf69c] process_one_work+0x25c/0x438<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc0000c00c8] worker_thread+0x32c/0x448<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc0000c4aa8] kthread+0xf8/0x100<br>
Feb 21 22:36:16 Sailfish kernel: ---[ end trace baf5d4897624fa10 ]---<br>
Feb 21 22:36:16 Sailfish kernel: qpnp-smbcharger qpnp-smbcharger-18: node /soc/qcom,spmi@200f000/qcom,pmi8950@2/qcom,qpnp-smbcharge<br>
r IO resource absent!<br>
Feb 21 22:36:16 Sailfish kernel: ------------[ cut here ]------------<br>
Feb 21 22:36:16 Sailfish kernel: WARNING: CPU: 0 PID: 79 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xiaom<br>
i/msm8953/drivers/usb/dwc3/dwc3-msm.c:3590 dwc3_otg_sm_work+0x268/0x610()<br>
Feb 21 22:36:16 Sailfish kernel: Modules linked in:<br>
Feb 21 22:36:16 Sailfish kernel: CPU: 0 PID: 79 Comm: kworker/0:1 Tainted: G        W      3.18.105-ElectraBlue-11.0-mido #3<br>
Feb 21 22:36:16 Sailfish kernel: Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
Feb 21 22:36:16 Sailfish kernel: Workqueue: events dwc3_otg_sm_work<br>
Feb 21 22:36:16 Sailfish kernel: Call trace:<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc00008a2c4] show_stack+0x20/0x28<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc0000a69ac] warn_slowpath_null+0x38/0x44<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc000774b58] dwc3_otg_sm_work+0x268/0x610<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc0000bf69c] process_one_work+0x25c/0x438<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc0000c00c8] worker_thread+0x32c/0x448<br>
Feb 21 22:36:16 Sailfish kernel: [ffffffc0000c4aa8] kthread+0xf8/0x100<br>
Feb 21 22:36:16 Sailfish kernel: ---[ end trace baf5d4897624fa11 ]---<br>
Feb 21 22:36:16 Sailfish kernel: set_usb_charge_mode_par off = 2<br>
Feb 21 22:36:16 Sailfish kernel: qpnp-smbcharger qpnp-smbcharger-18: SMBCHG successfully probe Charger version=SCHG_LITE Revision DIG:0.0<br>
 ANA:0.1 batt=1 dc=0 usb=0<br>
Feb 21 22:36:16 Sailfish kernel: EDAC DEVICE0: Giving out device to module soc:arm64-cpu-erp controller cache: DEV soc:arm64-cpu-erp (INT<br>
ERRUPT)<br>
Feb 21 22:36:16 Sailfish kernel: ARM64 CPU ERP: Could not find cci-irq IRQ property. Proceeding anyway.<br>
Feb 21 22:36:16 Sailfish kernel: ARM64 CPU ERP: SBE detection is disabled.<br>
Feb 21 22:36:16 Sailfish kernel: Registered cp15_barrier emulation handler<br>
Feb 21 22:36:16 Sailfish kernel: Registered setend emulation handler<br>
Feb 21 22:36:16 Sailfish kernel: Loading compiled-in X.509 certificates<br>
Feb 21 22:36:16 Sailfish kernel: Key type encrypted registered<br>
Feb 21 22:36:16 Sailfish kernel: msm-dwc3 7000000.ssusb: DWC3 in low power mode<br>
Feb 21 22:36:16 Sailfish kernel: fastrpc soc:qcom,adsprpc-mem: for adsp_rh segments only will be dumped.<br>
Feb 21 22:36:16 Sailfish kernel: RNDIS_IPA module is loaded.<br>
Feb 21 22:36:16 Sailfish kernel: file system registered<br>
Feb 21 22:36:16 Sailfish kernel: mbim_init: initialize 1 instances<br>
Feb 21 22:36:16 Sailfish kernel: mbim_init: Initialized 1 ports<br>
Feb 21 22:36:16 Sailfish kernel: rndis_qc_init: initialize rndis QC instance<br>
Feb 21 22:36:16 Sailfish kernel: Number of LUNs=8<br>
Feb 21 22:36:16 Sailfish kernel: Mass Storage Function, version: 2009/09/11<br>
Feb 21 22:36:16 Sailfish kernel: LUN: removable file: (no medium)<br>
Feb 21 22:36:16 Sailfish kernel: Number of LUNs=1<br>
Feb 21 22:36:16 Sailfish kernel: LUN: removable file: (no medium)<br>
Feb 21 22:36:16 Sailfish kernel: Number of LUNs=1<br>
Feb 21 22:36:16 Sailfish kernel: android_usb gadget: android_usb ready<br>
Feb 21 22:36:16 Sailfish kernel: input: gpio-keys as /devices/soc/soc:gpio_keys/input/input5<br>
Feb 21 22:36:16 Sailfish kernel: qcom,qpnp-rtc qpnp-rtc-8: setting system clock to 1970-02-21 20:36:09 UTC (4480569)<br>
Feb 21 22:36:16 Sailfish kernel: pwm-ir soc:pwm_ir: reg-id = vdd, low-active = 0, use-timer = 0<br>
Feb 21 22:36:16 Sailfish kernel: Registered IR keymap rc-lirc<br>
Feb 21 22:36:16 Sailfish kernel: input: pwm-ir as /devices/soc/soc:pwm_ir/rc/rc0/input6<br>
Feb 21 22:36:16 Sailfish kernel: rc0: pwm-ir as /devices/soc/soc:pwm_ir/rc/rc0<br>
Feb 21 22:36:16 Sailfish kernel: rc rc0: lirc_dev: driver ir-lirc-codec (pwm-ir) registered at minor = 0<br>
Feb 21 22:36:16 Sailfish kernel: msm-core initialized without polling period<br>
Feb 21 22:36:16 Sailfish kernel: parse_cpu_levels: idx 1 276<br>
Feb 21 22:36:16 Sailfish kernel: calculate_residency: residency  0 for LPM<br>
Feb 21 22:36:16 Sailfish kernel: parse_cpu_levels: idx 1 286<br>
Feb 21 22:36:16 Sailfish kernel: calculate_residency: residency  0 for LPM<br>
Feb 21 22:36:16 Sailfish kernel: qcom,qpnp-flash-led qpnp-flash-led-25: Unable to acquire pinctrl<br>
Feb 21 22:36:16 Sailfish kernel: rmnet_ipa started initialization<br>
Feb 21 22:36:16 Sailfish kernel: IPA SSR support = True<br>
Feb 21 22:36:16 Sailfish kernel: IPA ipa-loaduC = True<br>
Feb 21 22:36:16 Sailfish kernel: IPA SG support = True<br>
Feb 21 22:36:16 Sailfish kernel: ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
Feb 21 22:36:16 Sailfish kernel: ipa ipa2_uc_state_check:301 uC is not loaded<br>
Feb 21 22:36:16 Sailfish kernel: rmnet_ipa completed initialization<br>
Feb 21 22:36:16 Sailfish kernel: qcom,cc-debug-8953 1874000.qcom,cc-debug: Registered Debug Mux successfully<br>
Feb 21 22:36:16 Sailfish kernel: msm8x16_wcd_spmi_probe(6272):slave ID = 0x1<br>
Feb 21 22:36:16 Sailfish kernel: msm8x16_wcd_spmi_probe(6272):slave ID = 0x1<br>
Feb 21 22:36:16 Sailfish kernel: msm8952-asoc-wcd c051000.sound: default codec configured<br>
Feb 21 22:36:16 Sailfish kernel: msm8952-asoc-wcd c051000.sound: ASoC: platform (null) not registered<br>
Feb 21 22:36:16 Sailfish kernel: msm8952-asoc-wcd c051000.sound: snd_soc_register_card failed (-517)<br>
Feb 21 22:36:16 Sailfish kernel: apc_mem_acc_corner: disabling<br>
Feb 21 22:36:16 Sailfish kernel: gfx_mem_acc_corner: disabling<br>
Feb 21 22:36:16 Sailfish kernel: adv_vreg: disabling<br>
Feb 21 22:36:16 Sailfish kernel: vdd_vreg: disabling<br>
Feb 21 22:36:16 Sailfish kernel: clock_late_init: Removing enables held for handed-off clocks<br>
Feb 21 22:36:16 Sailfish kernel: ALSA device list:<br>
Feb 21 22:36:16 Sailfish kernel:   No soundcards found.<br>
Feb 21 22:36:16 Sailfish kernel: Freeing unused kernel memory: 892K<br>
Feb 21 22:36:16 Sailfish kernel: Freeing alternatives memory: 84K<br>
Feb 21 22:36:16 Sailfish kernel: EXT3-fs (mmcblk0p49): error: couldn't mount because of unsupported optional features (40)<br>
Feb 21 22:36:16 Sailfish kernel: EXT2-fs (mmcblk0p49): error: couldn't mount because of unsupported optional features (40)<br>
Feb 21 22:36:16 Sailfish kernel: EXT4-fs (mmcblk0p49): warning: maximal mount count reached, running e2fsck is recommended<br>
Feb 21 22:36:16 Sailfish kernel: EXT4-fs (mmcblk0p49): mounted filesystem with ordered data mode. Opts: (null)<br>
Feb 21 22:36:16 Sailfish kernel: enable_store: android_usb: already disabled<br>
Feb 21 22:36:16 Sailfish kernel: dwc3 7000000.dwc3: ep0out: Unable to dequeue while in LPM<br>
Feb 21 22:36:16 Sailfish kernel: rndis_function_bind_config: rndis_function_bind_config MAC: 00:00:00:00:00:00<br>
Feb 21 22:36:16 Sailfish kernel: android_usb gadget: using random self ethernet address<br>
Feb 21 22:36:16 Sailfish kernel: android_usb gadget: using random host ethernet address<br>
Feb 21 22:36:16 Sailfish kernel: rndis0: MAC d2:15:58:dd:84:20<br>
Feb 21 22:36:16 Sailfish kernel: rndis0: HOST MAC ee:60:50:97:8f:8c<br>
Feb 21 22:36:16 Sailfish kernel: hid keyboard<br>
Feb 21 22:36:16 Sailfish kernel: hidg_bind: creating device ffffffc0ac33c600<br>
Feb 21 22:36:16 Sailfish kernel: hid mouse<br>
Feb 21 22:36:16 Sailfish kernel: hidg_bind: creating device ffffffc0ac33c800<br>
Feb 21 22:36:16 Sailfish kernel: IPv6: ADDRCONF(NETDEV_UP): rndis0: link is not ready<br>
Feb 21 22:36:16 Sailfish kernel: of_batterydata_get_best_profile: qrd_msm8953_sunwoda_atl_4100mah found<br>
Feb 21 22:36:16 Sailfish kernel: FG: fg_batt_profile_init: Battery SOC: 100, V: 4211553uV<br>
Feb 21 22:36:16 Sailfish kernel: of_batterydata_get_best_profile: qrd_msm8953_sunwoda_atl_4100mah found<br>
Feb 21 22:36:16 Sailfish kernel: dwc3 7000000.dwc3: ep0out: Unable to dequeue while in LPM<br>
Feb 21 22:36:16 Sailfish kernel: hidg_unbind: destroying device ffffffc0ac33c600<br>
Feb 21 22:36:16 Sailfish kernel: hidg_unbind: destroying device ffffffc0ac33c800<br>
Feb 21 22:36:16 Sailfish kernel: [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
Feb 21 22:36:16 Sailfish kernel: [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
Feb 21 22:36:16 Sailfish kernel: rndis_function_bind_config: rndis_function_bind_config MAC: EE:60:50:97:8F:8C<br>
Feb 21 22:36:16 Sailfish kernel: android_usb gadget: using random self ethernet address<br>
Feb 21 22:36:16 Sailfish kernel: android_usb gadget: using previous host ethernet address<br>
Feb 21 22:36:16 Sailfish kernel: rndis0: MAC 6a:af:46:1c:92:cd<br>
Feb 21 22:36:16 Sailfish kernel: rndis0: HOST MAC ee:60:50:97:8f:8c<br>
Feb 21 22:36:16 Sailfish kernel: hid keyboard<br>
Feb 21 22:36:16 Sailfish kernel: hidg_bind: creating device ffffffc0af5e2e00<br>
Feb 21 22:36:16 Sailfish kernel: hid mouse<br>
Feb 21 22:36:16 Sailfish kernel: hidg_bind: creating device ffffffc0af5e2a00<br>
Feb 21 22:36:16 Sailfish kernel: dwc3 7000000.dwc3: ep0out: Unable to dequeue while in LPM<br>
Feb 21 22:36:16 Sailfish kernel: hidg_unbind: destroying device ffffffc0af5e2e00<br>
Feb 21 22:36:16 Sailfish kernel: hidg_unbind: destroying device ffffffc0af5e2a00<br>
Feb 21 22:36:16 Sailfish kernel: [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
Feb 21 22:36:16 Sailfish kernel: [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
Feb 21 22:36:16 Sailfish kernel: rndis_function_bind_config: rndis_function_bind_config MAC: EE:60:50:97:8F:8C<br>
Feb 21 22:36:16 Sailfish kernel: android_usb gadget: using random self ethernet address<br>
Feb 21 22:36:16 Sailfish kernel: android_usb gadget: using previous host ethernet address<br>
Feb 21 22:36:16 Sailfish kernel: rndis0: MAC 0a:91:b7:c6:ca:2d<br>
Feb 21 22:36:16 Sailfish kernel: rndis0: HOST MAC ee:60:50:97:8f:8c<br>
Feb 21 22:36:16 Sailfish kernel: hid keyboard<br>
Feb 21 22:36:16 Sailfish kernel: hidg_bind: creating device ffffffc0ac33cc00<br>
Feb 21 22:36:16 Sailfish kernel: hid mouse<br>
Feb 21 22:36:16 Sailfish kernel: hidg_bind: creating device ffffffc0ac33ce00<br>
Feb 21 22:36:16 Sailfish kernel: random: nonblocking pool is initialized<br>
Feb 21 22:36:16 Sailfish kernel: dwc3 7000000.dwc3: ep0out: Unable to dequeue while in LPM<br>
Feb 21 22:36:16 Sailfish kernel: hidg_unbind: destroying device ffffffc0ac33cc00<br>
Feb 21 22:36:16 Sailfish kernel: hidg_unbind: destroying device ffffffc0ac33ce00<br>
Feb 21 22:36:16 Sailfish kernel: [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
Feb 21 22:36:16 Sailfish kernel: [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
Feb 21 22:36:16 Sailfish kernel: rndis_function_bind_config: rndis_function_bind_config MAC: EE:60:50:97:8F:8C<br>
Feb 21 22:36:16 Sailfish kernel: android_usb gadget: using random self ethernet address<br>
Feb 21 22:36:16 Sailfish kernel: android_usb gadget: using previous host ethernet address<br>
Feb 21 22:36:16 Sailfish kernel: rndis0: MAC 56:82:1f:8c:57:89<br>
Feb 21 22:36:16 Sailfish kernel: rndis0: HOST MAC ee:60:50:97:8f:8c<br>
Feb 21 22:36:16 Sailfish kernel: hid keyboard<br>
Feb 21 22:36:16 Sailfish kernel: hidg_bind: creating device ffffffc0af5e2e00<br>
Feb 21 22:36:16 Sailfish kernel: hid mouse<br>
Feb 21 22:36:16 Sailfish kernel: hidg_bind: creating device ffffffc0af5e3200<br>
Feb 21 22:36:16 Sailfish kernel: dwc3 7000000.dwc3: ep0out: Unable to dequeue while in LPM<br>
Feb 21 22:36:16 Sailfish kernel: hidg_unbind: destroying device ffffffc0af5e2e00<br>
Feb 21 22:36:16 Sailfish kernel: hidg_unbind: destroying device ffffffc0af5e3200<br>
Feb 21 22:36:16 Sailfish kernel: [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
Feb 21 22:36:16 Sailfish kernel: [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
Feb 21 22:36:16 Sailfish kernel: rndis_function_bind_config: rndis_function_bind_config MAC: EE:60:50:97:8F:8C<br>
Feb 21 22:36:16 Sailfish kernel: android_usb gadget: using random self ethernet address<br>
Feb 21 22:36:16 Sailfish kernel: android_usb gadget: using previous host ethernet address<br>
Feb 21 22:36:16 Sailfish kernel: rndis0: MAC 86:1a:51:93:c0:33<br>
Feb 21 22:36:16 Sailfish kernel: rndis0: HOST MAC ee:60:50:97:8f:8c<br>
Feb 21 22:36:16 Sailfish kernel: hid keyboard<br>
Feb 21 22:36:16 Sailfish kernel: hidg_bind: creating device ffffffc0ae9b9600<br>
Feb 21 22:36:16 Sailfish kernel: hid mouse<br>
Feb 21 22:36:16 Sailfish kernel: hidg_bind: creating device ffffffc0ae9b9800<br>
Feb 21 22:36:16 Sailfish kernel: IPv6: ADDRCONF(NETDEV_UP): rndis0: link is not ready<br>
Feb 21 22:36:16 Sailfish kernel: EXT4-fs (mmcblk0p49): re-mounted. Opts: (null)<br>
Feb 21 22:36:16 Sailfish preinit: (15.85) Welcome to Sailfish OS 3.0.2.8 (Oulanka)<br>
Feb 21 22:36:16 Sailfish preinit: (15.86) is_erase_needed: No : 0<br>
Feb 21 22:36:16 Sailfish preinit: (15.88) get_bootstate: USER : 0<br>
Feb 21 22:36:16 Sailfish preinit: (15.88) BOOTSTATE = USER<br>
Feb 21 22:36:16 Sailfish preinit: (15.88) Booting to default.target<br>
Feb 21 22:36:16 Sailfish systemd[1]: systemd 225 running in system mode. (+PAM -AUDIT -SELINUX +IMA -APPARMOR +SMACK -SYSVINIT +UTMP -LIB<br>
CRYPTSETUP +GCRYPT -GNUTLS +ACL +XZ -LZ4 -SECCOMP +BLKID -ELFUTILS +KMOD -IDN)<br>
Feb 21 22:36:16 Sailfish systemd[1]: Detected architecture arm64.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Set hostname to Sailfish.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Initializing machine ID from random generator.<br>
Feb 21 22:36:16 Sailfish systemd[1]: [/lib/systemd/system/camera-hal.service:8] Executable path is not absolute, ignoring: setprop<br>
persist.camera.HAL3.enabled 0<br>
Feb 21 22:36:16 Sailfish systemd[1]: camera-hal.service: Service lacks both ExecStart= and ExecStop= setting. Refusing.<br>
Feb 21 22:36:16 Sailfish systemd[1]: sys-fs-pstore.mount: Cannot create mount unit for API file system /sys/fs/pstore. Refusing.<br>
Feb 21 22:36:16 Sailfish systemd[1]: sys-fs-pstore.mount: Cannot add dependency job, ignoring: Unit sys-fs-pstore.mount failed to l<br>
oad: Invalid argument. See system logs and 'systemctl status sys-fs-pstore.mount' for details.<br>
Feb 21 22:36:16 Sailfish systemd[1]: camera-hal.service: Cannot add dependency job, ignoring: Unit camera-hal.service failed to loa<br>
d: Invalid argument. See system logs and 'systemctl status camera-hal.service' for details.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Started Dispatch Password Requests to Console Directory Watch.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Started Forward Password Requests to Wall Directory Watch.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Reached target Swap.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Reached target Login Prompts.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Created slice Root Slice.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Listening on udev Kernel Socket.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Listening on Journal Audit Socket.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Listening on udev Control Socket.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Listening on Journal Socket (/dev/log).<br>
Feb 21 22:36:16 Sailfish systemd[1]: Listening on /dev/initctl Compatibility Named Pipe.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Created slice User and Session Slice.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Listening on Journal Socket.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Created slice System Slice.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Reached target Slices.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Mounting Droid mount for /dev/cpuset...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Mounting Debug File System...<br>
Feb 21 22:36:16 Sailfish kernel: cgroup: new mount options do not match the existing superblock, will be ignored<br>
Feb 21 22:36:16 Sailfish systemd[1]: Mounting Temporary Directory...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Starting Remount Root and Kernel File Systems...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Mounting FFS mount...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Starting Create list of required static device nodes for the current kernel...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Starting Journal Service...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Starting Setup Virtual Console...<br>
Feb 21 22:36:16 Sailfish kernel: EXT4-fs (mmcblk0p49): re-mounted. Opts: (null)<br>
Feb 21 22:36:16 Sailfish systemd[1]: Mounted Debug File System.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Mounted FFS mount.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Mounted Temporary Directory.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Mounted Droid mount for /dev/cpuset.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Started Create list of required static device nodes for the current kernel.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Started Setup Virtual Console.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Started Remount Root and Kernel File Systems.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Starting Create System Users...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Starting Rebuild Dynamic Linker Cache...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Starting Rebuild Hardware Database...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Starting Clean RPM db region files at each reboot...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Starting Load/Save Random Seed...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Started Create System Users.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Started Clean RPM db region files at each reboot.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Started Load/Save Random Seed.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Starting Create Static Device Nodes in /dev...<br>
Feb 21 22:36:16 Sailfish systemd-journal[686]: Journal started<br>
Feb 21 22:36:16 Sailfish systemd[1]: Started Journal Service.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Starting Flush Journal to Persistent Storage...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Started Create Static Device Nodes in /dev.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Reached target Local File Systems (Pre).<br>
Feb 21 22:36:16 Sailfish systemd[1]: Mounting Droid mount for /mnt...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Mounting Droid mount for /config...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Starting udev Kernel Device Manager...<br>
Feb 21 22:36:16 Sailfish systemd[1]: Mounted Droid mount for /mnt.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Mounted Droid mount for /config.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Started Flush Journal to Persistent Storage.<br>
Feb 21 22:36:16 Sailfish systemd-udevd[712]: unknown key 'DEVLINKS' in /lib/udev/rules.d/999-android-system.rules:741<br>
Feb 21 22:36:16 Sailfish systemd-udevd[712]: invalid rule '/lib/udev/rules.d/999-android-system.rules:741'<br>
Feb 21 22:36:16 Sailfish systemd-udevd[712]: unknown key 'DEVLINKS' in /lib/udev/rules.d/999-android-system.rules:746<br>
Feb 21 22:36:16 Sailfish systemd-udevd[712]: invalid rule '/lib/udev/rules.d/999-android-system.rules:746'<br>
Feb 21 22:36:16 Sailfish systemd-udevd[712]: unknown key 'DEVLINKS' in /lib/udev/rules.d/999-android-system.rules:751<br>
Feb 21 22:36:16 Sailfish systemd-udevd[712]: invalid rule '/lib/udev/rules.d/999-android-system.rules:751'<br>
Feb 21 22:36:16 Sailfish systemd[1]: Started udev Kernel Device Manager.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Started Rebuild Dynamic Linker Cache.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Started Rebuild Hardware Database.<br>
Feb 21 22:36:16 Sailfish systemd[1]: Starting udev Coldplug all Devices...<br>
Feb 21 22:36:16 Sailfish systemd-udevd[712]: unknown key 'DEVLINKS' in /lib/udev/rules.d/999-android-system.rules:741<br>
Feb 21 22:36:16 Sailfish systemd-udevd[712]: invalid rule '/lib/udev/rules.d/999-android-system.rules:741'<br>
Feb 21 22:36:16 Sailfish systemd-udevd[712]: unknown key 'DEVLINKS' in /lib/udev/rules.d/999-android-system.rules:746<br>
Feb 21 22:36:16 Sailfish systemd-udevd[712]: invalid rule '/lib/udev/rules.d/999-android-system.rules:746'<br>
Feb 21 22:36:16 Sailfish systemd-udevd[712]: unknown key 'DEVLINKS' in /lib/udev/rules.d/999-android-system.rules:751<br>
Feb 21 22:36:16 Sailfish systemd-udevd[712]: invalid rule '/lib/udev/rules.d/999-android-system.rules:751'<br>
Feb 21 22:36:17 Sailfish vgchange[1330]: /dev/mmcblk0rpmb: read failed after 0 of 4096 at 0: Input/output error<br>
Feb 21 22:36:17 Sailfish vgchange[1330]: /dev/mmcblk0rpmb: read failed after 0 of 4096 at 4128768: Input/output error<br>
Feb 21 22:36:17 Sailfish vgchange[1330]: /dev/mmcblk0rpmb: read failed after 0 of 4096 at 4186112: Input/output error<br>
Feb 21 22:36:17 Sailfish vgchange[1330]: /dev/mmcblk0rpmb: read failed after 0 of 4096 at 4096: Input/output error<br>
Feb 21 22:36:17 Sailfish systemd[1]: Started udev Coldplug all Devices.<br>
Feb 21 22:36:17 Sailfish systemd[1]: Starting udev Wait for Complete Device Initialization...<br>
Feb 21 22:36:17 Sailfish systemd[1]: Found device /dev/mmcblk0p12.<br>
Feb 21 22:36:17 Sailfish systemd[1]: Mounting Droid mount for /dsp...<br>
Feb 21 22:36:17 Sailfish systemd[1]: Found device /dev/mmcblk0p1.<br>
Feb 21 22:36:17 Sailfish systemd[1]: Found device /dev/mmcblk0p26.<br>
Feb 21 22:36:17 Sailfish kernel: EXT4-fs (mmcblk0p12): mounted filesystem with ordered data mode. Opts: (null)<br>
Feb 21 22:36:17 Sailfish systemd[1]: Mounted Droid mount for /dsp.<br>
Feb 21 22:36:17 Sailfish systemd[1]: Found device /dev/mmcblk0p24.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Mounting Droid mount for /system...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Mounting Droid mount for /persist...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Mounting Droid mount for /firmware...<br>
Feb 21 22:36:18 Sailfish kernel: EXT4-fs (mmcblk0p26): mounted filesystem with ordered data mode. Opts: (null)<br>
Feb 21 22:36:18 Sailfish kernel: EXT4-fs (mmcblk0p24): mounted filesystem with ordered data mode. Opts: barrier=1<br>
Feb 21 22:36:18 Sailfish systemd[1]: Mounted Droid mount for /system.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Mounted Droid mount for /persist.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Mounted Droid mount for /firmware.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Reached target Local File Systems.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Rebuild Journal Catalog...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Create Volatile Files and Directories...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Listening on D-Bus System Message Bus Socket.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting StateFS FUSE filesystem, system-wide...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting D-Bus System Message Bus...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Enable Bluetooth HCI over SMD...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Listening on Nemo device lock socket.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Create sshd host keys...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Mode Control Entity (MCE)...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Mounting Configuration File System...<br>
Feb 21 22:36:18 Sailfish sshd-hostkeys[2272]: Generating ed25519 key: /etc/ssh/ssh_host_ed25519_key<br>
Feb 21 22:36:18 Sailfish systemd[1]: Mounting FUSE Control File System...<br>
Feb 21 22:36:18 Sailfish sshd-hostkeys[2272]: Generating rsa key: /etc/ssh/ssh_host_rsa_key<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Apply Kernel Variables...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Mounted Configuration File System.<br>
Feb 21 22:36:18 Sailfish kernel: Loading modules backported from Linux version next-20160324-0-g6f30d29<br>
Feb 21 22:36:18 Sailfish kernel: Backport generated by backports.git backports-20160324-0-g7dba139<br>
Feb 21 22:36:18 Sailfish systemd[1]: Mounted FUSE Control File System.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started Rebuild Journal Catalog.<br>
Feb 21 22:36:18 Sailfish mce[2275]: mce.c: main(): MCE 1.99.8 (release) starting up<br>
Feb 21 22:36:18 Sailfish systemd-sysctl[2358]: Couldn't write 'fq_codel' to 'net/core/default_qdisc', ignoring: No such file or directory<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started Create Volatile Files and Directories.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started StateFS FUSE filesystem, system-wide.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started D-Bus System Message Bus.<br>
Feb 21 22:36:18 Sailfish kernel: Bluetooth: Core ver 2.21<br>
Feb 21 22:36:18 Sailfish kernel: NET: Registered protocol family 31<br>
Feb 21 22:36:18 Sailfish kernel: Bluetooth: HCI device and connection manager initialized<br>
Feb 21 22:36:18 Sailfish kernel: Bluetooth: HCI socket layer initialized<br>
Feb 21 22:36:18 Sailfish kernel: Bluetooth: L2CAP socket layer initialized<br>
Feb 21 22:36:18 Sailfish kernel: Bluetooth: SCO socket layer initialized<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started udev Wait for Complete Device Initialization.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started Apply Kernel Variables.<br>
Feb 21 22:36:18 Sailfish sh[2583]: could not set property<br>
Feb 21 22:36:18 Sailfish sh[2583]: could not set property<br>
Feb 21 22:36:18 Sailfish sh[2583]: could not set property<br>
Feb 21 22:36:18 Sailfish rfkill[2589]: unblock set for all<br>
Feb 21 22:36:18 Sailfish kernel: hcismd_set_enable 0<br>
Feb 21 22:36:18 Sailfish kernel: Bluetooth: Cannot open the command channel<br>
Feb 21 22:36:18 Sailfish mce[2275]: modules/display.c: g_module_check_init(): display state req: ON<br>
Feb 21 22:36:18 Sailfish mce[2275]: modules/display.c: mdy_display_state_leave(): current display state = POWER_UP<br>
Feb 21 22:36:18 Sailfish mce[2275]: modules/display.c: mdy_stm_step(): forced brightness sync to: 254<br>
Feb 21 22:36:18 Sailfish mce[2275]: modules/display.c: mdy_stm_set_compositor_availability_changed(): compositor availability chang<br>
e: handled<br>
Feb 21 22:36:18 Sailfish mce[2275]: modules/proximity.c: report_proximity(): state: OPEN - CLOSED<br>
Feb 21 22:36:18 Sailfish mce[2275]: powerkey.c: xngf_create_client(): can't use ngfd - service not running<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started Mode Control Entity (MCE).<br>
Feb 21 22:36:18 Sailfish mce[2275]: modules/battery-udev.c: mcebat_update(): charger_state: undefined - off<br>
Feb 21 22:36:18 Sailfish mce[2275]: modules/battery-udev.c: mcebat_update(): battery_status: undefined - ok<br>
Feb 21 22:36:18 Sailfish mce[2275]: modules/display.c: mdy_display_state_enter(): current display state = ON<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting usb-moded USB gadget controller...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting ohm daemon for resource policy management...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting droid-hal-init...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Get initial bootstate...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Update UTMP about System Boot/Shutdown...<br>
Feb 21 22:36:18 Sailfish usb_moded[2592]: usb_moded 0.86.0+mer30 starting<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Update is Completed...<br>
Feb 21 22:36:18 Sailfish get-initial-bootstate[2595]: Initial BOOTSTATE=USER<br>
Feb 21 22:36:18 Sailfish droid-hal-init: init first stage started!<br>
Feb 21 22:36:18 Sailfish droid-hal-init: init second stage started!<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started Get initial bootstate.<br>
Feb 21 22:36:18 Sailfish usb_moded[2592]: CONFIGFS not detected<br>
Feb 21 22:36:18 Sailfish usb_moded[2592]: ANDROID0 detected<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started Update is Completed.<br>
Feb 21 22:36:18 Sailfish kernel: dwc3 7000000.dwc3: ep0out: Unable to dequeue while in LPM<br>
Feb 21 22:36:18 Sailfish kernel: hidg_unbind: destroying device ffffffc0ae9b9600<br>
Feb 21 22:36:18 Sailfish kernel: hidg_unbind: destroying device ffffffc0ae9b9800<br>
Feb 21 22:36:18 Sailfish kernel: [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
Feb 21 22:36:18 Sailfish droid-hal-init: Failed to initialize property area<br>
Feb 21 22:36:18 Sailfish droid-hal-init: Running restorecon...<br>
Feb 21 22:36:18 Sailfish droid-hal-init: waitpid failed: No child processes<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Loading properties from /default.prop took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /init.environ.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: /init.qcom.rc: 636: invalid keyword 'system'<br>
Feb 21 22:36:18 Sailfish droid-hal-init: init.target.rc: 176: invalid keyword 'load_all_props'<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing init.target.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /init.qcom.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /init.usb.configfs.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /init.zygote64_32.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /init.cm.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /init.rc took 0.01s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: Waiting for /dev/.coldboot_done...<br>
Feb 21 22:36:18 Sailfish droid-hal-init: Waiting for /dev/.coldboot_done took 0.00s.<br>
Feb 21 22:36:18 Sailfish droid-hal-init: /dev/hw_random not found<br>
Feb 21 22:36:18 Sailfish kernel: keychord: using input dev qpnp_pon for fevent<br>
Feb 21 22:36:18 Sailfish kernel: keychord: using input dev ft5435_ts for fevent<br>
Feb 21 22:36:18 Sailfish kernel: keychord: using input dev s2w_pwrkey for fevent<br>
Feb 21 22:36:18 Sailfish kernel: keychord: using input dev gf3208 for fevent<br>
Feb 21 22:36:18 Sailfish kernel: keychord: using input dev gpio-keys for fevent<br>
Feb 21 22:36:18 Sailfish droid-hal-init: loglevel: invalid log level'64'<br>
Feb 21 22:36:18 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/kernel/hung_task_timeout_secs': No such file or dire<br>
ctorye--<br>
Feb 21 22:36:18 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/foreground/cpus': Permission denied<br>
Feb 21 22:36:18 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/foreground/mems': Permission denied<br>
Feb 21 22:36:18 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/foreground/boost/cpus': Permission denied<br>
Feb 21 22:36:18 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/foreground/boost/mems': Permission denied<br>
Feb 21 22:36:18 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/background/cpus': Permission denied<br>
Feb 21 22:36:18 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/background/mems': Permission denied<br>
Feb 21 22:36:18 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/system-background/cpus': Permission denied<br>
Feb 21 22:36:18 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/system-background/mems': Permission denied<br>
Feb 21 22:36:18 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/top-app/cpus': Permission denied<br>
Feb 21 22:36:18 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/top-app/mems': Permission denied<br>
Feb 21 22:36:18 Sailfish kernel: Registered swp emulation handler<br>
Feb 21 22:36:18 Sailfish droid-hal-init: /dev/hw_random not found<br>
Feb 21 22:36:18 Sailfish droid-hal-init: Starting service 'droid_init_done'...<br>
Feb 21 22:36:18 Sailfish fs_mgr: Cannot open file fstab.qcom<br>
Feb 21 22:36:18 Sailfish droid-hal-init: fs_mgr_mount_all returned an error<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started Update UTMP about System Boot/Shutdown.<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/atrace.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/bootanim.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish kernel: [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister rndis0<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/bootstat.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/debuggerd.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/drmserver.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/dumpstate.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/gatekeeperd.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/init-debug.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/installd.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/logcatd.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/logd.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/mediacodec.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/mediadrmserver.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/mediaextractor.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/mtpd.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/perfprofd.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/racoon.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/rild.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/servicemanager.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: Could not import file '/usr/libexec/droid-hybris/system/etc/init/superuser.rc'<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/uncrypt.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Parsing /usr/libexec/droid-hybris/system/etc/init/vdc.rc took 0.00s.)<br>
Feb 21 22:36:18 Sailfish systemd[1]: Reached target System Initialization.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Listening on OpenSSH Server Socket.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Reached target Sockets.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started Wayland path watcher.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Reached target Paths.<br>
Feb 21 22:36:18 Sailfish droid-hal-init: fs_mgr_mount_all returned unexpected error 255<br>
Feb 21 22:36:18 Sailfish droid-hal-init: Starting service 'logd'...<br>
Feb 21 22:36:18 Sailfish droid-hal-init: couldn't write 2616 to /dev/cpuset/system-background/tasks: No space left on device<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started Daily Cleanup of Temporary Directories.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Reached target Timers.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting DSME...<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Loading properties from /system/build.prop took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Loading properties from /vendor/build.prop took 0.00s.)<br>
Feb 21 22:36:18 Sailfish droid-hal-init: (Loading properties from /factory/factory.prop took 0.00s.)<br>
Feb 21 22:36:18 Sailfish fs_mgr: Cannot open file /fstab.qcom<br>
Feb 21 22:36:18 Sailfish droid-hal-init: unable to read fstab /fstab.qcom: No such file or directory<br>
Feb 21 22:36:18 Sailfish droid-hal-init: Starting service 'debuggerd'...<br>
Feb 21 22:36:18 Sailfish droid-hal-init: do_start: Service debuggerd64 not found<br>
Feb 21 22:36:18 Sailfish droid-hal-init: do_start: Service vold not found<br>
Feb 21 22:36:18 Sailfish droid-hal-init: couldn't write 2618 to /dev/cpuset/system-background/tasks: No space left on device<br>
Feb 21 22:36:18 Sailfish dsme[2617]: DSME 0.79.3 starting up<br>
Feb 21 22:36:18 Sailfish droid-hal-init: Not bootcharting.<br>
Feb 21 22:36:18 Sailfish dsme[2617]: dsme wdd: Could not open any watchdog filesdsme wdd: no WD's opened; WD kicking disabled<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started usb-moded USB gadget controller.<br>
Feb 21 22:36:18 Sailfish systemd[1]: droid-hal-init.service: Supervising process 2600 which is not our child. We'll most likely not<br>
 notice when it exits.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started droid-hal-init.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Reached target Basic System.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Started droid-late-start.<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Disk Manager...<br>
Feb 21 22:36:18 Sailfish DSME[2619]: state: new state: USER<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Link Android folder to home...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Login Service...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Load wifi module...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Indicate boot is done...<br>
Feb 21 22:36:18 Sailfish systemd[1]: Starting Nemo device lock daemon...<br>
Feb 21 22:36:18 Sailfish kernel: wcnss_wlan triggered by userspace<br>
Feb 21 22:36:18 Sailfish kernel: wcnss_pm_qos_add_request: add request<br>
Feb 21 22:36:18 Sailfish kernel: wcnss_pm_qos_update_request: update request 100<br>
Feb 21 22:36:18 Sailfish kernel: wcnss_notif_cb: wcnss notification event: 2<br>
Mar 12 23:24:24 Sailfish DSME[2619]: IPHB: rtc delta to 1547945286<br>
Mar 12 23:24:24 Sailfish systemd[1]: Starting Telephony service...<br>
Mar 12 23:24:24 Sailfish udisksd[2629]: udisks daemon version 2.8.1 starting<br>
Mar 12 23:24:24 Sailfish systemd[1]: Started Link Android folder to home.<br>
Mar 12 23:24:24 Sailfish android-links.sh[2630]: /usr/bin/droid/android-links.sh: line 2: [: missing `]'<br>
Mar 12 23:24:24 Sailfish kernel: subsys-pil-tz a21b000.qcom,pronto: wcnss: loading from 0x000000008e700000 to 0x000000008ed42000<br>
Mar 12 23:24:24 Sailfish kernel: wcnss_notif_cb: wcnss notification event: 6<br>
Mar 12 23:24:24 Sailfish kernel: wcnss: IRIS Reg: 04000004<br>
Mar 12 23:24:24 Sailfish audit: CONFIG_CHANGE audit_pid=2616 old=0 auid=4294967295 ses=4294967295 subj=kernel res=1<br>
Mar 12 23:24:24 Sailfish audit: CONFIG_CHANGE audit_rate_limit=20 old=0 auid=4294967295 ses=4294967295 subj=kernel res=1<br>
Mar 12 23:24:24 Sailfish kernel: audit: type=1305 audit(1552425864.019:2): audit_pid=2616 old=0 auid=4294967295 ses=4294967295 subj<br>
=kernel res=1<br>
Mar 12 23:24:24 Sailfish logd.auditd: start<br>
Mar 12 23:24:24 Sailfish logd.klogd: 19140765095<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: oFono version 1.21<br>
Mar 12 23:24:24 Sailfish systemd[1]: Started Login Service.<br>
Mar 12 23:24:24 Sailfish ohmd[2593]: E: accessories: failed to open device '/dev/input/event7'<br>
Mar 12 23:24:24 Sailfish systemd-logind[2632]: New seat seat0.<br>
Mar 12 23:24:24 Sailfish dbus[2443]: [system] Activating via systemd: service name='org.freedesktop.PolicyKit1' unit='dbus-org.free<br>
desktop.PolicyKit1.service'<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding udev hardware detection<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding Novatel modem driver<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding Sierra modem driver<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding ZTE modem driver<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding Icera modem driver<br>
Mar 12 23:24:24 Sailfish systemd[1]: sys-fs-pstore.mount: Cannot add dependency job, ignoring: Unit sys-fs-pstore.mount failed to l<br>
oad: Invalid argument. See system logs and 'systemctl status sys-fs-pstore.mount' for details.<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding Huawei modem driver<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding Calypso modem driver<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding MBM modem driver<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding Telit modem driver<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding HSO modem driver<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding Infineon modem driver<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding STE modem driver<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding Dialup modem driver<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding Hands-Free Profile Driver<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding SpeedUp modem driver<br>
Mar 12 23:24:24 Sailfish systemd[1]: Started Telephony service.<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding Phone Simulator driver<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding CDMA AT modem driver<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding External Hands-Free Profile Plugin<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding Dial-up Networking Profile Plugins<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Excluding CDMA provisioning Plugin<br>
Mar 12 23:24:24 Sailfish systemd[1]: Starting Authorization Manager...<br>
Mar 12 23:24:24 Sailfish audit[2658]: ANOM_ABEND auid=4294967295 uid=0 gid=0 ses=4294967295 subj=kernel pid=2658 comm="vdc" exe="/system/<br>
bin/vdc" sig=6<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'exec 1 (/system/bin/tzdatacheck)'...<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Invalid dataCallFormat config value (13)<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: Invalid dataCallFormat config value (13)<br>
Mar 12 23:24:24 Sailfish polkitd[2659]: started daemon version 0.105 using authority implementation `local' version `0.105'<br>
Mar 12 23:24:24 Sailfish kernel: capability: warning: `ofonod' uses 32-bit capabilities (legacy support in use)<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Service 'exec 1 (/system/bin/tzdatacheck)' (pid 2663) exited with status 0<br>
Mar 12 23:24:24 Sailfish dbus[2443]: [system] Successfully activated service 'org.freedesktop.PolicyKit1'<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: [grilio-socket] ERROR! Can't connect to RILD: No such file or directory<br>
Mar 12 23:24:24 Sailfish systemd[1]: Started Authorization Manager.<br>
Mar 12 23:24:24 Sailfish ofonod[2637]: [grilio-socket] ERROR! Can't connect to RILD: No such file or directory<br>
Mar 12 23:24:24 Sailfish unknown: type=1305 audit(1552425864.019:3): audit_rate_limit=20 old=0 auid=4294967295 ses=4294967295 subj=<br>
kernel res=1<br>
Mar 12 23:24:24 Sailfish unknown: type=1701 audit(1552425864.055:4): auid=4294967295 uid=0 gid=0 ses=4294967295 subj=kernel pid=265<br>
8 comm="vdc" exe="/system/bin/vdc" sig=6<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'perfd'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2670 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/sys/block/dm-0/queue/read_ahead_kb': No such file or director<br>
y<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/sys/block/dm-1/queue/read_ahead_kb': No such file or director<br>
y<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'sysinit'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: (Loading properties from /data/local.prop took 0.00s.)<br>
Mar 12 23:24:24 Sailfish systemd[1]: Started Nemo device lock daemon.<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'logd-reinit'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2708 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish kernel: subsys-pil-tz c200000.qcom,lpass: adsp: loading from 0x000000008d600000 to 0x000000008e700000<br>
Mar 12 23:24:24 Sailfish logd.daemon: reinit<br>
Mar 12 23:24:24 Sailfish systemd[1]: Started DSME.<br>
Mar 12 23:24:24 Sailfish systemd[1]: Starting Oneshot stuff for root...<br>
Mar 12 23:24:24 Sailfish kernel: subsys-pil-tz a21b000.qcom,pronto: wcnss: Brought out of reset<br>
Mar 12 23:24:24 Sailfish DSME[2619]: thermal sensor generic: No thermal config files found<br>
Mar 12 23:24:24 Sailfish oneshot[2715]: oneshot: /etc/oneshot.d/0/dconf-update - OK<br>
Mar 12 23:24:24 Sailfish oneshot[2715]: oneshot: /etc/oneshot.d/0/msyncd-storage-perm - OK<br>
Mar 12 23:24:24 Sailfish oneshot[2715]: oneshot: /etc/oneshot.d/0/signon-storage-perm - OK<br>
Mar 12 23:24:24 Sailfish dbus[2443]: [system] Activating via systemd: service name='org.nemo.ssu' unit='dbus-org.nemo.ssu.service'<br>
<br>
Mar 12 23:24:24 Sailfish systemd[1]: sys-fs-pstore.mount: Cannot add dependency job, ignoring: Unit sys-fs-pstore.mount failed to l<br>
oad: Invalid argument. See system logs and 'systemctl status sys-fs-pstore.mount' for details.<br>
Mar 12 23:24:24 Sailfish systemd[1]: Starting SSU service...<br>
Mar 12 23:24:24 Sailfish dbus[2443]: [system] Successfully activated service 'org.nemo.ssu'<br>
Mar 12 23:24:24 Sailfish systemd[1]: Started SSU service.<br>
Mar 12 23:24:24 Sailfish oneshot[2715]: oneshot: /etc/oneshot.d/0/ssu-update-repos - OK<br>
Mar 12 23:24:24 Sailfish systemd[1]: Started Oneshot stuff for root.<br>
Mar 12 23:24:24 Sailfish systemd[1]: Started ohm daemon for resource policy management.<br>
Mar 12 23:24:24 Sailfish systemd[1]: Starting Connection service...<br>
Mar 12 23:24:24 Sailfish systemd[1]: Starting Sensor daemon for sensor framework...<br>
Mar 12 23:24:24 Sailfish kernel: subsys-pil-tz c200000.qcom,lpass: adsp: Brought out of reset<br>
Mar 12 23:24:24 Sailfish systemd[1]: Starting Permit User Sessions...<br>
Mar 12 23:24:24 Sailfish systemd[1]: Started Permit User Sessions.<br>
Mar 12 23:24:24 Sailfish systemd[1]: Starting Start User Session...<br>
Mar 12 23:24:24 Sailfish connmand[2806]: Connection Manager version 1.32+git61<br>
Mar 12 23:24:24 Sailfish start-autologin[2810]: Starting autologin for 100000<br>
Mar 12 23:24:24 Sailfish connmand[2806]: Cannot restore table security, file /var/lib/connman/iptables/security.v4 not found<br>
Mar 12 23:24:24 Sailfish connmand[2806]: Cannot restore table raw, file /var/lib/connman/iptables/raw.v4 not found<br>
Mar 12 23:24:24 Sailfish connmand[2806]: Cannot restore table nat, file /var/lib/connman/iptables/nat.v4 not found<br>
Mar 12 23:24:24 Sailfish connmand[2806]: Cannot restore table mangle, file /var/lib/connman/iptables/mangle.v4 not found<br>
Mar 12 23:24:24 Sailfish connmand[2806]: Cannot restore table filter, file /var/lib/connman/iptables/filter.v4 not found<br>
Mar 12 23:24:24 Sailfish audit: NETFILTER_CFG table=filter family=2 entries=4<br>
Mar 12 23:24:24 Sailfish audit[2806]: SYSCALL arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40 a3=2e5868 items=0 ppid=1 pid=2<br>
806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/conn<br>
mand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish audit: AUDIT1327 proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B747<br>
2616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish systemd[1]: Started Sensor daemon for sensor framework.<br>
Mar 12 23:24:24 Sailfish unknown: type=1325 audit(1552425864.555:5): table=filter family=2 entries=4<br>
Mar 12 23:24:24 Sailfish unknown: type=1300 audit(1552425864.555:5): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40 a<br>
3=2e5868 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="<br>
connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish unknown: type=1327 audit(1552425864.555:5): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E<br>
6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish unknown: type=1320 audit(1552425864.555:5):<br>
Mar 12 23:24:24 Sailfish audit: NETFILTER_CFG table=mangle family=2 entries=6<br>
Mar 12 23:24:24 Sailfish audit[2806]: SYSCALL arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40 a3=2e5b18 items=0 ppid=1 pid=2<br>
806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/conn<br>
mand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish audit: AUDIT1327 proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B747<br>
2616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish unknown: type=1325 audit(1552425864.559:6): table=mangle family=2 entries=6<br>
Mar 12 23:24:24 Sailfish unknown: type=1300 audit(1552425864.559:6): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40 a<br>
3=2e5b18 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="<br>
connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish unknown: type=1327 audit(1552425864.559:6): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E<br>
6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish unknown: type=1320 audit(1552425864.559:6):<br>
Mar 12 23:24:24 Sailfish systemd[1]: sys-fs-pstore.mount: Cannot add dependency job, ignoring: Unit sys-fs-pstore.mount failed to l<br>
oad: Invalid argument. See system logs and 'systemctl status sys-fs-pstore.mount' for details.<br>
Mar 12 23:24:24 Sailfish kernel: subsys-pil-tz c200000.qcom,lpass: Subsystem error monitoring/handling services are up<br>
Mar 12 23:24:24 Sailfish kernel: subsys-pil-tz c200000.qcom,lpass: adsp: Power/Clock ready interrupt received<br>
Mar 12 23:24:24 Sailfish kernel: L-Notify: Generel: 7<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Service 'sysinit' (pid 2671) exited with status 0<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Service 'logd-reinit' (pid 2708) exited with status 0<br>
Mar 12 23:24:24 Sailfish kernel: sensors-ssc soc:qcom,msm-ssc-sensors: slpi_loader_do: pil get failed,<br>
Mar 12 23:24:24 Sailfish kernel: sensors-ssc soc:qcom,msm-ssc-sensors: slpi_loader_do: SLPI image loading failed<br>
Mar 12 23:24:24 Sailfish kernel: diag: In diag_send_feature_mask_update, control channel is not open, p: 1, 0000000000000000<br>
Mar 12 23:24:24 Sailfish kernel: msm8x16_wcd_spmi_probe(6272):slave ID = 0x1<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'irsc_util'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'rmt_storage'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'tftp_server'...<br>
Mar 12 23:24:24 Sailfish unknown: type=1325 audit(1552425864.567:7): table=nat family=2 entries=5<br>
Mar 12 23:24:24 Sailfish audit: NETFILTER_CFG table=nat family=2 entries=5<br>
Mar 12 23:24:24 Sailfish audit[2806]: SYSCALL arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40 a3=2e59c0 items=0 ppid=1 pid=2<br>
806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/conn<br>
mand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish audit: AUDIT1327 proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B747<br>
2616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish systemd[1]: Created slice system-autologin.slice.<br>
Mar 12 23:24:24 Sailfish kernel: msm8x16_wcd_spmi_probe(6272):slave ID = 0x1<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'config_bt_addr'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'config_bluetooth'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'sensors'...<br>
Mar 12 23:24:24 Sailfish systemd[1]: Started Autologin user 100000.<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'gx_fpd'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'per_mgr'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'servicemanager'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2841 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'cnd'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'netmgrd'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2835 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'qti'...<br>
Mar 12 23:24:24 Sailfish start-autologin[2810]: autologin\@100000.service started<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2834 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'ril-daemon2'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2842 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2840 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'thermal-engine'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'wcnss-service'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'imsqmidaemon'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2845 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'adsprpcd'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'hvdcp_opti'...<br>
Mar 12 23:24:24 Sailfish unknown: type=1300 audit(1552425864.567:7): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40 a<br>
3=2e59c0 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="<br>
connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish unknown: type=1327 audit(1552425864.567:7): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E<br>
6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'energy-awareness'...<br>
Mar 12 23:24:24 Sailfish unknown: type=1320 audit(1552425864.567:7):<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2855 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'drm'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'installd'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2859 to /dev/cpuset/foreground/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish unknown: type=1325 audit(1552425864.587:8): table=filter family=10 entries=4<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2847 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2846 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2848 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish audit: NETFILTER_CFG table=filter family=10 entries=4<br>
Mar 12 23:24:24 Sailfish audit[2806]: SYSCALL arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=29 a2=40 a3=2e5a28 items=0 ppid=1 pid=<br>
2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/con<br>
nmand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish audit: AUDIT1327 proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B747<br>
2616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish kernel: msm8952-asoc-wcd c051000.sound: default codec configured<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2853 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish unknown: type=1300 audit(1552425864.587:8): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=29 a2=40<br>
a3=2e5a28 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm=<br>
"connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'mediacodec'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'mediadrm'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'mediaextractor'...<br>
Mar 12 23:24:24 Sailfish kernel: msm8952-asoc-wcd c051000.sound: ASoC: platform (null) not registered<br>
Mar 12 23:24:24 Sailfish kernel: msm8952-asoc-wcd c051000.sound: snd_soc_register_card failed (-517)<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'ril-daemon'...<br>
Mar 12 23:24:24 Sailfish unknown: type=1327 audit(1552425864.587:8): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E<br>
6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish unknown: type=1320 audit(1552425864.587:8):<br>
Mar 12 23:24:24 Sailfish kernel: msm8952-asoc-wcd c051000.sound: default codec configured<br>
Mar 12 23:24:24 Sailfish kernel: wcd-spmi-core msm8x16_wcd_codec-11: Error: regulator not found:cdc-vdd-spkdrv<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2867 to /dev/cpuset/foreground/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2866 to /dev/cpuset/foreground/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2865 to /dev/cpuset/foreground/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'minimedia'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'minisf'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'miniaf'...<br>
Mar 12 23:24:24 Sailfish audit: NETFILTER_CFG table=mangle family=10 entries=6<br>
Mar 12 23:24:24 Sailfish audit[2806]: SYSCALL arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=29 a2=40 a3=2e5dc0 items=0 ppid=1 pid=<br>
2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/con<br>
nmand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish audit: AUDIT1327 proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B747<br>
2616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish unknown: type=1325 audit(1552425864.643:9): table=mangle family=10 entries=6<br>
Mar 12 23:24:24 Sailfish unknown: type=1300 audit(1552425864.643:9): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=29 a2=40<br>
a3=2e5dc0 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm=<br>
"connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish unknown: type=1327 audit(1552425864.643:9): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E<br>
6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish unknown: type=1320 audit(1552425864.643:9):<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Service 'irsc_util' (pid 2832) exited with status 0<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Service 'config_bt_addr' (pid 2836) exited with status 0<br>
Mar 12 23:24:24 Sailfish wait-session-to-start[2852]: In wait loop (0) - state: unknown<br>
Mar 12 23:24:24 Sailfish systemd[2844]: pam_unix(autologin:account): account nemo has password changed in future<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet0/accept_ra': No such file or dir<br>
ectory<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no dapm match for MM_DL5 -- MultiMedia5 -- TERT_<br>
MI2S_RX Audio Mixer<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route MM_DL5 - MultiMedia5 - TERT_<br>
MI2S_RX Audio Mixer<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no dapm match for MM_DL8 -- MultiMedia8 -- TERT_<br>
MI2S_RX Audio Mixer<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route MM_DL8 - MultiMedia8 - TERT_<br>
MI2S_RX Audio Mixer<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet1/accept_ra': No such file or dir<br>
ectory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet2/accept_ra': No such file or dir<br>
ectory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet3/accept_ra': No such file or dir<br>
ectory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet4/accept_ra': No such file or dir<br>
ectory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet5/accept_ra': No such file or dir<br>
ectory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet6/accept_ra': No such file or dir<br>
ectory<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no dapm match for INT_FM_TX -- INTERNAL_FM_TX --<br>
 SEC_MI2S_RX Port Mixer<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route INT_FM_TX - INTERNAL_FM_TX -<br>
 SEC_MI2S_RX Port Mixer<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet7/accept_ra': No such file or dir<br>
ectory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio0/accept_ra': No such file o<br>
r directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio1/accept_ra': No such file o<br>
r directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio2/accept_ra': No such file o<br>
r directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio3/accept_ra': No such file o<br>
r directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio4/accept_ra': No such file o<br>
r directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio5/accept_ra': No such file o<br>
r directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio6/accept_ra': No such file o<br>
r directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_sdio7/accept_ra': No such file o<br>
r directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_usb0/accept_ra': No such file or<br>
 directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_usb1/accept_ra': No such file or<br>
 directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_usb2/accept_ra': No such file or<br>
 directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/proc/sys/net/ipv6/conf/rmnet_usb3/accept_ra': No such file or<br>
 directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: sanitize_path: Failed to locate path /dev/block/bootdevice/by-name/rawdump: No such file o<br>
r directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: do_chown: sanitize_path failed for path: /dev/block/bootdevice/by-name/rawdump prefix:/dev<br>
/block/<br>
Mar 12 23:24:24 Sailfish droid-hal-init: sanitize_path: Failed to locate path /dev/block/bootdevice/by-name/rawdump: No such file o<br>
r directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: do_chmod: failed for /dev/block/bootdevice/by-name/rawdump..prefix(/dev/block/) match err<br>
<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/top-app/cpus': Permission denied<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/top-app/boost/cpus': No such file or directory<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/foreground/cpus': Permission denied<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/foreground/boost/cpus': Permission denied<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no dapm match for VOICEMMODE1_DL -- VoiceMMode1 -<br>
- SEC_MI2S_RX_Voice Mixer<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route VOICEMMODE1_DL - VoiceMMode1<br>
- SEC_MI2S_RX_Voice Mixer<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no dapm match for VOICEMMODE2_DL -- VoiceMMode2 -<br>
- SEC_MI2S_RX_Voice Mixer<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route VOICEMMODE2_DL - VoiceMMode2<br>
- SEC_MI2S_RX_Voice Mixer<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no source widget found for AUDIO_REF_EC_UL3 MUX<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route AUDIO_REF_EC_UL3 MUX - direct<br>
 - MM_UL3<br>
Mar 12 23:24:24 Sailfish unknown: type=1006 audit(1552425864.675:10): pid=2844 uid=0 subj=kernel old-auid=4294967295 auid=100000 ol<br>
d-ses=4294967295 ses=1 res=1<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no dapm match for SEC_MI2S_TX -- SEC_MI2S_TX --<br>
SLIMBUS_0_RX Port Mixer<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route SEC_MI2S_TX - SEC_MI2S_TX -<br>
SLIMBUS_0_RX Port Mixer<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/background/cpus': Permission denied<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/system-background/cpus': Permission denied<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no dapm match for VOICE2_STUB_DL -- Voice2 Stub -<br>
- INTERNAL_BT_SCO_RX_Voice Mixer<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route VOICE2_STUB_DL - Voice2 Stub<br>
- INTERNAL_BT_SCO_RX_Voice Mixer<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/camera-daemon/cpus': Permission denied<br>
Mar 12 23:24:24 Sailfish droid-hal-init: write_file: Unable to open '/dev/cpuset/camera-daemon/mems': Permission denied<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: no sink widget found for SENARY_MI2S_TX<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to add route BE_IN - direct - SENARY_MI2S<br>
_TX<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'dpmd'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'loc_launcher'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'qcom-sh'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'qseeproxydaemon'...<br>
Mar 12 23:24:24 Sailfish kernel: sysmon-qmi: sysmon_clnt_svc_arrive: Connection established between QMI handle and adsp's SSCTL service<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'qcamerasvr'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2931 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish systemd[1]: Started Create sshd host keys.<br>
Mar 12 23:24:24 Sailfish systemd[2844]: pam_systemd(autologin:session): Using 600s D-Bus method call timeout<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2924 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'audiod'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'gatekeeperd'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2925 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2930 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'perfprofd'...<br>
Mar 12 23:24:24 Sailfish unknown: type=1325 audit(1552425864.727:11): table=filter family=10 entries=4<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'console'...<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'imsdatadaemon'...<br>
Mar 12 23:24:24 Sailfish audit: NETFILTER_CFG table=filter family=10 entries=4<br>
Mar 12 23:24:24 Sailfish audit[2806]: SYSCALL arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=29 a2=40 a3=2e6dd0 items=0 ppid=1 pid=<br>
2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/con<br>
nmand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish audit: AUDIT1327 proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B747<br>
2616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'per_proxy'...<br>
Mar 12 23:24:24 Sailfish systemd[1]: Created slice user-100000.slice.<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2946 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2953 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2954 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish droid-hal-init: couldn't write 2951 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:24 Sailfish systemd[1]: sys-fs-pstore.mount: Cannot add dependency job, ignoring: Unit sys-fs-pstore.mount failed to l<br>
oad: Invalid argument. See system logs and 'systemctl status sys-fs-pstore.mount' for details.<br>
Mar 12 23:24:24 Sailfish unknown: type=1300 audit(1552425864.727:11): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=29 a2=40<br>
 a3=2e6dd0 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm<br>
="connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish unknown: type=1327 audit(1552425864.727:11): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006<br>
E6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish unknown: type=1320 audit(1552425864.727:11):<br>
Mar 12 23:24:24 Sailfish kernel: subsys-pil-tz a21b000.qcom,pronto: wcnss: Power/Clock ready interrupt received<br>
Mar 12 23:24:24 Sailfish kernel: wcnss_notif_cb: wcnss notification event: 7<br>
Mar 12 23:24:24 Sailfish audit: NETFILTER_CFG table=filter family=10 entries=8<br>
Mar 12 23:24:24 Sailfish audit[2806]: SYSCALL arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=29 a2=40 a3=2e7308 items=0 ppid=1 pid=<br>
2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/con<br>
nmand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish audit: AUDIT1327 proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B747<br>
2616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish unknown: type=1325 audit(1552425864.767:12): table=filter family=10 entries=8<br>
Mar 12 23:24:24 Sailfish unknown: type=1300 audit(1552425864.767:12): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=29 a2=40<br>
 a3=2e7308 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm<br>
="connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish unknown: type=1327 audit(1552425864.767:12): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006<br>
E6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish unknown: type=1320 audit(1552425864.767:12):<br>
Mar 12 23:24:24 Sailfish kernel: subsys-pil-tz a21b000.qcom,pronto: Subsystem error monitoring/handling services are up<br>
Mar 12 23:24:24 Sailfish kernel: wcnss_notif_cb: wcnss notification event: 3<br>
Mar 12 23:24:24 Sailfish kernel: wcnss_pm_qos_update_request: update request -1<br>
Mar 12 23:24:24 Sailfish kernel: wcnss_pm_qos_remove_request: remove request<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: Failed to create SEC_MI2S_RX Port Mixer debugfs fi<br>
le<br>
Mar 12 23:24:24 Sailfish kernel: hcismd_set_enable 1<br>
Mar 12 23:24:24 Sailfish kernel: Bluetooth: Cannot open the command channel<br>
Mar 12 23:24:24 Sailfish kernel: apr_tal:Q6 Is Up<br>
Mar 12 23:24:24 Sailfish kernel: 'opened /dev/adsprpc-smd c 227 0'<br>
Mar 12 23:24:24 Sailfish unknown: type=1325 audit(1552425864.795:13): table=filter family=2 entries=4<br>
Mar 12 23:24:24 Sailfish rfkill[2917]: unblock set for all<br>
Mar 12 23:24:24 Sailfish audit: NETFILTER_CFG table=filter family=2 entries=4<br>
Mar 12 23:24:24 Sailfish audit[2806]: SYSCALL arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40 a3=2e6b00 items=0 ppid=1 pid=2<br>
806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/conn<br>
mand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish audit: AUDIT1327 proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B747<br>
2616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish kernel: diag: In diag_send_feature_mask_update, control channel is not open, p: 2, 0000000000000000<br>
Mar 12 23:24:24 Sailfish unknown: type=1300 audit(1552425864.795:13): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40<br>
a3=2e6b00 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm=<br>
"connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish unknown: type=1327 audit(1552425864.795:13): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006<br>
E6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish unknown: type=1320 audit(1552425864.795:13):<br>
Mar 12 23:24:24 Sailfish audit: NETFILTER_CFG table=filter family=2 entries=8<br>
Mar 12 23:24:24 Sailfish audit[2806]: SYSCALL arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40 a3=2e6f60 items=0 ppid=1 pid=2<br>
806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="connmand" exe="/usr/sbin/conn<br>
mand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish audit: AUDIT1327 proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006E6C3830323131002D2D6E6F6261636B747<br>
2616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish unknown: type=1325 audit(1552425864.831:14): table=filter family=2 entries=8<br>
Mar 12 23:24:24 Sailfish unknown: type=1300 audit(1552425864.831:14): arch=40000028 syscall=294 success=yes exit=0 a0=9 a1=0 a2=40<br>
a3=2e6f60 items=0 ppid=1 pid=2806 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm=<br>
"connmand" exe="/usr/sbin/connmand" subj=kernel key=(null)<br>
Mar 12 23:24:24 Sailfish unknown: type=1327 audit(1552425864.831:14): proctitle=2F7573722F7362696E2F636F6E6E6D616E64002D6E002D57006<br>
E6C3830323131002D2D6E6F6261636B7472616365002D2D73797374656D64002D2D6E6F706C7567696E3D77696669<br>
Mar 12 23:24:24 Sailfish unknown: type=1320 audit(1552425864.831:14):<br>
Mar 12 23:24:24 Sailfish kernel: msm-pcm-routing soc:qcom,msm-pcm-routing: ASoC: mux SLIM_0_RX AANC MUX has no paths<br>
Mar 12 23:24:24 Sailfish kernel: wcd-spmi-core msm8x16_wcd_codec-11: ASoC: mux RX3 MIX1 INP3 has no paths<br>
Mar 12 23:24:24 Sailfish kernel: wcd-spmi-core msm8x16_wcd_codec-11: ASoC: mux RX2 MIX1 INP3 has no paths<br>
Mar 12 23:24:24 Sailfish kernel: msm_thermal:set_enabled enabled = 0<br>
Mar 12 23:24:24 Sailfish kernel: audit: audit_lost=1 audit_rate_limit=20 audit_backlog_limit=64<br>
Mar 12 23:24:24 Sailfish kernel: audit: rate limit exceeded<br>
Mar 12 23:24:24 Sailfish unknown: type=1325 audit(1552425864.859:15): table=filter family=10 entries=9<br>
Mar 12 23:24:24 Sailfish audit: NETFILTER_CFG table=filter family=10 entries=9<br>
Mar 12 23:24:24 Sailfish systemd[1]: Started Load wifi module.<br>
Mar 12 23:24:24 Sailfish kernel: subsys-restart: __subsystem_get(): Changing subsys fw_name to modem<br>
Mar 12 23:24:24 Sailfish kernel: IPA received MPSS BEFORE_POWERUP<br>
Mar 12 23:24:24 Sailfish kernel: ipa ipa2_uc_state_check:301 uC is not loaded<br>
Mar 12 23:24:24 Sailfish kernel: ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
Mar 12 23:24:24 Sailfish kernel: IPA BEFORE_POWERUP handling is complete<br>
Mar 12 23:24:24 Sailfish kernel: msm8952-asoc-wcd c051000.sound: control 3:0:0:IEC958 Playback PCM Stream:0 is already present<br>
Mar 12 23:24:24 Sailfish kernel: msm-hdmi-dba-codec-rx soc:qcom,msm-hdmi-dba-codec-rx: ASoC: Failed to add IEC958 Playback PCM Stre<br>
am: -16<br>
Mar 12 23:24:24 Sailfish kernel: msm_dba_get_probed_device: Device not found (adv7533, 0)<br>
Mar 12 23:24:24 Sailfish kernel: msm_dba_register_client: Device not found (adv7533, 0)<br>
Mar 12 23:24:24 Sailfish kernel: msm_hdmi_dba_codec_rx_init_dba: error in registering audio client 0<br>
Mar 12 23:24:24 Sailfish kernel: wcnss_wlan_ctrl_probe: SMD ctrl channel up<br>
Mar 12 23:24:24 Sailfish kernel: wcnss: version 01050102<br>
Mar 12 23:24:24 Sailfish kernel: wcnss: schedule dnld work for pronto<br>
Mar 12 23:24:24 Sailfish kernel: wcnss: build version CNSS-PR-4-0-00328<br>
Mar 12 23:24:24 Sailfish droid-hal-init: Starting service 'msm_irqbalance'...<br>
Mar 12 23:24:25 Sailfish kernel: wlan: module is from the staging directory, the quality is unknown, you have been warned.<br>
Mar 12 23:24:25 Sailfish kernel: pil-q6v5-mss 4080000.qcom,mss: modem: loading from 0x0000000086c00000 to 0x000000008c200000<br>
Mar 12 23:24:25 Sailfish systemd-logind[2632]: New session 1 of user nemo.<br>
Mar 12 23:24:25 Sailfish kernel: input: msm8953-snd-card-mtp Headset Jack as /devices/soc/c051000.sound/sound/card0/input7<br>
Mar 12 23:24:25 Sailfish kernel: input: msm8953-snd-card-mtp Button Jack as /devices/soc/c051000.sound/sound/card0/input8<br>
Mar 12 23:24:25 Sailfish udisksd[2629]: Acquired the name org.freedesktop.UDisks2 on the system message bus<br>
Mar 12 23:24:25 Sailfish systemd[1]: Started Disk Manager.<br>
Mar 12 23:24:25 Sailfish systemd-udevd[712]: unknown key 'DEVLINKS' in /lib/udev/rules.d/999-android-system.rules:741<br>
Mar 12 23:24:25 Sailfish kernel: wlan: loading driver v3.0.11.66<br>
Mar 12 23:24:25 Sailfish systemd-udevd[712]: invalid rule '/lib/udev/rules.d/999-android-system.rules:741'<br>
Mar 12 23:24:25 Sailfish systemd-udevd[712]: unknown key 'DEVLINKS' in /lib/udev/rules.d/999-android-system.rules:746<br>
Mar 12 23:24:25 Sailfish systemd-udevd[712]: invalid rule '/lib/udev/rules.d/999-android-system.rules:746'<br>
Mar 12 23:24:25 Sailfish systemd-udevd[712]: unknown key 'DEVLINKS' in /lib/udev/rules.d/999-android-system.rules:751<br>
Mar 12 23:24:25 Sailfish systemd-udevd[712]: invalid rule '/lib/udev/rules.d/999-android-system.rules:751'<br>
Mar 12 23:24:25 Sailfish systemd[1]: Reached target Sound Card.<br>
Mar 12 23:24:25 Sailfish systemd[1]: Started Session 1 of user nemo.<br>
Mar 12 23:24:25 Sailfish systemd[1]: Starting User Manager for UID 100000...<br>
Mar 12 23:24:25 Sailfish droid-hal-init: Service 'droid_init_done' (pid 2611) exited with status 0<br>
Mar 12 23:24:25 Sailfish droid-hal-init: Service 'config_bluetooth' (pid 2837) exited with status 0<br>
Mar 12 23:24:25 Sailfish systemd[3245]: pam_unix(systemd-user:account): account nemo has password changed in future<br>
Mar 12 23:24:25 Sailfish systemd[3245]: pam_unix(systemd-user:session): session opened for user nemo by (uid=0)<br>
Mar 12 23:24:25 Sailfish validate-user[3245]: /usr/lib/startup/validate-user checking  100000<br>
Mar 12 23:24:25 Sailfish connmand[2806]: Creation of /var/lib/connman/global_proxy watch failed<br>
Mar 12 23:24:25 Sailfish dbus[2443]: [system] Activating via systemd: service name='net.connman.vpn' unit='connman-vpn.service'<br>
Mar 12 23:24:25 Sailfish systemd[3266]: pam_unix(systemd-user:session): session closed for user nemo<br>
Mar 12 23:24:25 Sailfish systemd[1]: sys-fs-pstore.mount: Cannot add dependency job, ignoring: Unit sys-fs-pstore.mount failed to l<br>
oad: Invalid argument. See system logs and 'systemctl status sys-fs-pstore.mount' for details.<br>
Mar 12 23:24:25 Sailfish systemd[3344]: pam_unix(systemd-user:account): account nemo has password changed in future<br>
Mar 12 23:24:25 Sailfish systemd[3344]: pam_unix(systemd-user:session): session opened for user nemo by (uid=0)<br>
Mar 12 23:24:25 Sailfish droid-hal-init: Service 'energy-awareness' (pid 2858) exited with status 0<br>
Mar 12 23:24:25 Sailfish dbus[2443]: [system] Activating via systemd: service name='org.freedesktop.hostname1' unit='dbus-org.freed<br>
esktop.hostname1.service'<br>
Mar 12 23:24:25 Sailfish systemd[1]: Started Connection service.<br>
Mar 12 23:24:25 Sailfish dbus[2443]: [system] Activating via systemd: service name='fi.w1.wpa_supplicant1' unit='wpa_supplicant.ser<br>
vice'<br>
Mar 12 23:24:25 Sailfish systemd[1]: sys-fs-pstore.mount: Cannot add dependency job, ignoring: Unit sys-fs-pstore.mount failed to l<br>
oad: Invalid argument. See system logs and 'systemctl status sys-fs-pstore.mount' for details.<br>
Mar 12 23:24:25 Sailfish systemd[1]: sys-fs-pstore.mount: Cannot add dependency job, ignoring: Unit sys-fs-pstore.mount failed to l<br>
oad: Invalid argument. See system logs and 'systemctl status sys-fs-pstore.mount' for details.<br>
Mar 12 23:24:25 Sailfish systemd[1]: Starting WPA Supplicant daemon...<br>
Mar 12 23:24:25 Sailfish systemd[1]: Starting Hostname Service...<br>
Mar 12 23:24:25 Sailfish systemd[1]: Starting ConnMan VPN service...<br>
Mar 12 23:24:25 Sailfish systemd[1]: Starting Bluetooth service...<br>
Mar 12 23:24:25 Sailfish kernel: pil-q6v5-mss 4080000.qcom,mss: Debug policy not present - msadp. Continue.<br>
Mar 12 23:24:25 Sailfish kernel: pil-q6v5-mss 4080000.qcom,mss: Loading MBA and DP (if present) from 0x00000000f4b00000 to 0x00000000f4c0<br>
0000 size 100000<br>
Mar 12 23:24:25 Sailfish sensorfwd[2807]: HYBRIS CTL setActive23=ORIENTATION, true) - -1=Unknown error -1<br>
Mar 12 23:24:25 Sailfish dbus[2443]: [system] Successfully activated service 'fi.w1.wpa_supplicant1'<br>
Mar 12 23:24:25 Sailfish systemd[1]: Started WPA Supplicant daemon.<br>
Mar 12 23:24:25 Sailfish wpa_supplicant[3398]: Successfully initialized wpa_supplicant<br>
Mar 12 23:24:25 Sailfish connman-vpnd[3406]: Connection Manager VPN daemon version 1.32+git61<br>
Mar 12 23:24:25 Sailfish dbus[2443]: [system] Successfully activated service 'net.connman.vpn'<br>
Mar 12 23:24:25 Sailfish systemd[1]: Started ConnMan VPN service.<br>
Mar 12 23:24:25 Sailfish dbus[2443]: [system] Successfully activated service 'org.freedesktop.hostname1'<br>
Mar 12 23:24:25 Sailfish systemd[1]: Started Hostname Service.<br>
Mar 12 23:24:25 Sailfish bluetoothd[3421]: bluetoothd[3421]: Bluetooth daemon 5.47<br>
Mar 12 23:24:25 Sailfish bluetoothd[3421]: Bluetooth daemon 5.47<br>
Mar 12 23:24:25 Sailfish ofonod[2637]: Register Profile interface failed: /bluetooth/profile/hfp_ag<br>
Mar 12 23:24:25 Sailfish bluetoothd[3421]: Starting SDP server<br>
Mar 12 23:24:25 Sailfish systemd[1]: Started Bluetooth service.<br>
Mar 12 23:24:25 Sailfish systemd[1]: Reached target Network.<br>
Mar 12 23:24:25 Sailfish bluetoothd[3421]: bluetoothd[3421]: Starting SDP server<br>
Mar 12 23:24:25 Sailfish kernel: Bluetooth: BNEP (Ethernet Emulation) ver 1.3<br>
Mar 12 23:24:25 Sailfish kernel: Bluetooth: BNEP filters: protocol multicast<br>
Mar 12 23:24:25 Sailfish kernel: Bluetooth: BNEP socket layer initialized<br>
Mar 12 23:24:25 Sailfish bluetoothd[3421]: Bluetooth management interface 1.12 initialized<br>
Mar 12 23:24:25 Sailfish bluetoothd[3421]: bluetoothd[3421]: Bluetooth management interface 1.12 initialized<br>
Mar 12 23:24:25 Sailfish kernel: pil-q6v5-mss 4080000.qcom,mss: MBA boot done<br>
Mar 12 23:24:25 Sailfish connmand[2806]: Method "ListAdapters" with signature "" on interface "org.bluez.Manager" doesn't exist<br>
Mar 12 23:24:25 Sailfish wait-session-to-start[2852]: In wait loop (1) - state: activating<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Reached target Sockets.<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Reached target Timers.<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Started Session root process booster socket path.<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Reached target Paths.<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Reached target Basic System.<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Reached target Booster session.<br>
Mar 12 23:24:25 Sailfish systemd[2844]: pam_unix(autologin:session): session opened for user nemo by (uid=0)<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Startup finished in 395ms.<br>
Mar 12 23:24:25 Sailfish systemd[2844]: pam_systemd(autologin:session): Using 60s D-Bus method call timeout<br>
Mar 12 23:24:25 Sailfish systemd[1]: Started User Manager for UID 100000.<br>
Mar 12 23:24:25 Sailfish systemd[2844]: pam_systemd(autologin:session): Cannot create session: Already running in a session<br>
Mar 12 23:24:25 Sailfish rfkill[3527]: unblock set for all<br>
Mar 12 23:24:25 Sailfish kernel: hcismd_set_enable 1<br>
Mar 12 23:24:25 Sailfish kernel: Bluetooth: Cannot open the command channel<br>
Mar 12 23:24:25 Sailfish /usr/libexec/mapplauncherd/booster-silica-session[2844]: Daemon: Boot mode set.<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Started OS update timer.<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Created slice -.slice.<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Starting Oneshot stuff for user...<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Starting StateFS FUSE filesystem...<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Listening on D-Bus Session Message Bus Socket.<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Starting OHM Session Agent...<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Starting Generic application launch booster...<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Starting Runs basic setup for the Jolla user before the rest of the user session is started...<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Starting Load initial part of start-up wizard before user session begins...<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Started D-Bus Session Message Bus.<br>
Mar 12 23:24:25 Sailfish kernel: QSEECOM: qseecom_load_app: App (goodixfp) does'nt exist, loading apps for first time<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Started OHM Session Agent.<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Starting Time Daemon...<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Starting Application launch booster for Qt5...<br>
Mar 12 23:24:25 Sailfish ofonod[2637]: [grilio-socket] ERROR! Can't connect to RILD: Connection refused<br>
Mar 12 23:24:25 Sailfish ofonod[2637]: [grilio-socket] ERROR! Can't connect to RILD: Connection refused<br>
Mar 12 23:24:25 Sailfish dbus-daemon[3547]: Activating via systemd: service name='com.nokia.profiled' unit='profiled.service'<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Started Generic application launch booster.<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Started StateFS FUSE filesystem.<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Starting Profile Daemon...<br>
Mar 12 23:24:25 Sailfish dbus-daemon[3547]: Activating service name='ca.desrt.dconf'<br>
Mar 12 23:24:25 Sailfish dbus-daemon[3547]: Successfully activated service 'ca.desrt.dconf'<br>
Mar 12 23:24:25 Sailfish /usr/libexec/mapplauncherd/booster-qt5[3558]: Daemon: Boot mode set.<br>
Mar 12 23:24:25 Sailfish audit: NETFILTER_CFG table=mangle family=2 entries=14<br>
Mar 12 23:24:25 Sailfish audit[3581]: SYSCALL arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=0 a2=40 a3=7fa4814800 items=0 ppid=284<br>
7 pid=3581 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system<br>
/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:25 Sailfish audit: AUDIT1327 proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D74006D616E676C65002D4E0071636F6D5F7<br>
16F735F72657365745F504F5354524F5554494E47<br>
Mar 12 23:24:25 Sailfish kernel: QSEECOM: qseecom_load_app: App with id 5 (goodixfp) now loaded<br>
Mar 12 23:24:25 Sailfish kernel: --------driver_init_partial start.--------<br>
Mar 12 23:24:25 Sailfish kernel: --------gf_parse_dts start.--------<br>
Mar 12 23:24:25 Sailfish kernel: goodix_fp soc:goodix_fp: goodix,gpio_reset 140<br>
Mar 12 23:24:25 Sailfish kernel: goodix_fp soc:goodix_fp: goodix,gpio_irq 48<br>
Mar 12 23:24:25 Sailfish kernel: msm8953-pinctrl 1000000.pinctrl: invalid group "gpio135" for function "blsp_spi6"<br>
Mar 12 23:24:25 Sailfish kernel: msm8953-pinctrl 1000000.pinctrl: invalid group "gpio136" for function "blsp_spi6"<br>
Mar 12 23:24:25 Sailfish kernel: msm8953-pinctrl 1000000.pinctrl: invalid group "gpio137" for function "blsp_spi6"<br>
Mar 12 23:24:25 Sailfish kernel: msm8953-pinctrl 1000000.pinctrl: invalid group "gpio138" for function "blsp_spi6"<br>
Mar 12 23:24:25 Sailfish kernel: found pin control goodixfp_reset_reset<br>
Mar 12 23:24:25 Sailfish kernel: found pin control goodixfp_reset_active<br>
Mar 12 23:24:25 Sailfish kernel: found pin control goodixfp_irq_active<br>
Mar 12 23:24:25 Sailfish kernel: goodix_fp soc:goodix_fp: Selected 'goodixfp_reset_active'<br>
Mar 12 23:24:25 Sailfish kernel: goodix_fp soc:goodix_fp: Selected 'goodixfp_irq_active'<br>
Mar 12 23:24:25 Sailfish kernel: --------gf_parse_dts end---OK.--------<br>
Mar 12 23:24:25 Sailfish kernel: ------------[ cut here ]------------<br>
Mar 12 23:24:25 Sailfish kernel: WARNING: CPU: 7 PID: 2841 at /parentroot/parentroot/data/piggz/mer/android/droid.mido14/kernel/xia<br>
omi/msm8953/kernel/irq/manage.c:448 __enable_irq+0x4c/0x90()<br>
Mar 12 23:24:25 Sailfish kernel: Unbalanced enable for IRQ 55<br>
Mar 12 23:24:25 Sailfish kernel: Modules linked in: bnep(O) wlan(C+) hci_smd(O) bluetooth(O) compat(O) skcipher(O) autofs4<br>
Mar 12 23:24:25 Sailfish kernel: CPU: 7 PID: 2841 Comm: gx_fpd Tainted: G        WC O   3.18.105-ElectraBlue-11.0-mido #3<br>
Mar 12 23:24:25 Sailfish kernel: Hardware name: Qualcomm Technologies, Inc. MSM8953 + PMI8950 QRD SKU3 (DT)<br>
Mar 12 23:24:25 Sailfish kernel: Call trace:<br>
Mar 12 23:24:25 Sailfish kernel: [ffffffc00008a058] dump_backtrace+0x0/0x24c<br>
Mar 12 23:24:25 Sailfish kernel: [ffffffc00008a2c4] show_stack+0x20/0x28<br>
Mar 12 23:24:25 Sailfish kernel: [ffffffc000e0a2a0] dump_stack+0x80/0xa4<br>
Mar 12 23:24:25 Sailfish kernel: [ffffffc0000a67f8] warn_slowpath_common+0x94/0xb8<br>
Mar 12 23:24:25 Sailfish kernel: [ffffffc0000a68a4] warn_slowpath_fmt+0x88/0xac<br>
Mar 12 23:24:25 Sailfish kernel: [ffffffc000102a18] __enable_irq+0x4c/0x90<br>
Mar 12 23:24:25 Sailfish kernel: [ffffffc000102ae0] enable_irq+0x84/0xac<br>
Mar 12 23:24:25 Sailfish kernel: [ffffffc00082eaa4] gf_ioctl+0x268/0x4e8<br>
Mar 12 23:24:25 Sailfish kernel: [ffffffc0001f2368] do_vfs_ioctl+0x4ec/0x5e0<br>
Mar 12 23:24:25 Sailfish kernel: [ffffffc0001f24c8] SyS_ioctl+0x6c/0x94<br>
Mar 12 23:24:25 Sailfish kernel: ---[ end trace baf5d4897624fa13 ]---<br>
Mar 12 23:24:25 Sailfish kernel: goodix_fp soc:goodix_fp: Selected 'goodixfp_reset_reset'<br>
Mar 12 23:24:25 Sailfish unknown: type=1325 audit(1552425865.959:47): table=mangle family=2 entries=14<br>
Mar 12 23:24:25 Sailfish unknown: type=1300 audit(1552425865.959:47): arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=0 a2=40<br>
a3=7fa4814800 items=0 ppid=2847 pid=3581 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=429496729<br>
5 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:25 Sailfish unknown: type=1327 audit(1552425865.959:47): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D7<br>
4006D616E676C65002D4E0071636F6D5F716F735F72657365745F504F5354524F5554494E47<br>
Mar 12 23:24:25 Sailfish unknown: type=1320 audit(1552425865.959:47):<br>
Mar 12 23:24:25 Sailfish kernel: goodix_fp soc:goodix_fp: Selected 'goodixfp_reset_active'<br>
Mar 12 23:24:25 Sailfish kernel: goodix_fp soc:goodix_fp: IRQ after reset 0<br>
Mar 12 23:24:25 Sailfish unknown: type=1325 audit(1552425865.971:48): table=mangle family=2 entries=16<br>
Mar 12 23:24:25 Sailfish unknown: type=1300 audit(1552425865.971:48): arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=0 a2=40<br>
a3=7fa6e15400 items=0 ppid=2847 pid=3590 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=429496729<br>
5 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:25 Sailfish unknown: type=1327 audit(1552425865.971:48): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D7<br>
4006D616E676C65002D4100504F5354524F5554494E47002D6A0071636F6D5F716F735F72657365745F504F5354524F5554494E47<br>
Mar 12 23:24:25 Sailfish unknown: type=1320 audit(1552425865.971:48):<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Started Application launch booster for Qt5.<br>
Mar 12 23:24:25 Sailfish audit: NETFILTER_CFG table=mangle family=2 entries=16<br>
Mar 12 23:24:25 Sailfish audit[3590]: SYSCALL arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=0 a2=40 a3=7fa6e15400 items=0 ppid=284<br>
7 pid=3590 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system<br>
/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:25 Sailfish audit: AUDIT1327 proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D74006D616E676C65002D4100504F5354524<br>
F5554494E47002D6A0071636F6D5F716F735F72657365745F504F5354524F5554494E47<br>
Mar 12 23:24:25 Sailfish systemd[3344]: Started Runs basic setup for the Jolla user before the rest of the user session is started.<br>
Mar 12 23:24:25 Sailfish audit: NETFILTER_CFG table=mangle family=10 entries=6<br>
Mar 12 23:24:25 Sailfish audit[3598]: SYSCALL arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=29 a2=40 a3=7f8fe3f000 items=0 ppid=28<br>
47 pid=3598 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="ip6tables" exe="/syst<br>
em/bin/ip6tables" subj=kernel key=(null)<br>
Mar 12 23:24:25 Sailfish audit: AUDIT1327 proctitle=2F73797374656D2F62696E2F6970367461626C6573002D77002D74006D616E676C65002D4E0071636F6D5<br>
F716F735F72657365745F504F5354524F5554494E47<br>
Mar 12 23:24:25 Sailfish unknown: type=1325 audit(1552425865.987:49): table=mangle family=10 entries=6<br>
Mar 12 23:24:25 Sailfish unknown: type=1300 audit(1552425865.987:49): arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=29 a2=40<br>
 a3=7f8fe3f000 items=0 ppid=2847 pid=3598 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=42949672<br>
95 comm="ip6tables" exe="/system/bin/ip6tables" subj=kernel key=(null)<br>
Mar 12 23:24:25 Sailfish unknown: type=1327 audit(1552425865.987:49): proctitle=2F73797374656D2F62696E2F6970367461626C6573002D77002<br>
D74006D616E676C65002D4E0071636F6D5F716F735F72657365745F504F5354524F5554494E47<br>
Mar 12 23:24:25 Sailfish unknown: type=1320 audit(1552425865.987:49):<br>
Mar 12 23:24:26 Sailfish audit: NETFILTER_CFG table=mangle family=10 entries=8<br>
Mar 12 23:24:26 Sailfish audit[3602]: SYSCALL arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=29 a2=40 a3=7f88847000 items=0 ppid=28<br>
47 pid=3602 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="ip6tables" exe="/syst<br>
em/bin/ip6tables" subj=kernel key=(null)<br>
Mar 12 23:24:26 Sailfish audit: AUDIT1327 proctitle=2F73797374656D2F62696E2F6970367461626C6573002D77002D74006D616E676C65002D4100504F53545<br>
24F5554494E47002D6A0071636F6D5F716F735F72657365745F504F5354524F5554494E47<br>
Mar 12 23:24:26 Sailfish dbus-daemon[3547]: Successfully activated service 'com.nokia.profiled'<br>
Mar 12 23:24:26 Sailfish systemd[3344]: Started Profile Daemon.<br>
Mar 12 23:24:26 Sailfish unknown: type=1325 audit(1552425866.007:50): table=mangle family=10 entries=8<br>
Mar 12 23:24:26 Sailfish unknown: type=1300 audit(1552425866.007:50): arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=29 a2=40<br>
 a3=7f88847000 items=0 ppid=2847 pid=3602 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=42949672<br>
95 comm="ip6tables" exe="/system/bin/ip6tables" subj=kernel key=(null)<br>
Mar 12 23:24:26 Sailfish unknown: type=1327 audit(1552425866.007:50): proctitle=2F73797374656D2F62696E2F6970367461626C6573002D77002<br>
D74006D616E676C65002D4100504F5354524F5554494E47002D6A0071636F6D5F716F735F72657365745F504F5354524F5554494E47<br>
Mar 12 23:24:26 Sailfish unknown: type=1320 audit(1552425866.007:50):<br>
Mar 12 23:24:26 Sailfish audit: NETFILTER_CFG table=mangle family=2 entries=17<br>
Mar 12 23:24:26 Sailfish audit[3609]: SYSCALL arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=0 a2=40 a3=7f9943b000 items=0 ppid=284<br>
7 pid=3609 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system<br>
/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:26 Sailfish audit: AUDIT1327 proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D74006D616E676C65002D4E0071636F6D5F7<br>
16F735F66696C7465725F504F5354524F5554494E47<br>
Mar 12 23:24:26 Sailfish unknown: type=1325 audit(1552425866.019:51): table=mangle family=2 entries=17<br>
Mar 12 23:24:26 Sailfish unknown: type=1300 audit(1552425866.019:51): arch=c00000b7 syscall=208 success=yes exit=0 a0=b a1=0 a2=40<br>
a3=7f9943b000 items=0 ppid=2847 pid=3609 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=429496729<br>
5 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:26 Sailfish unknown: type=1327 audit(1552425866.019:51): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D7<br>
4006D616E676C65002D4E0071636F6D5F716F735F66696C7465725F504F5354524F5554494E47<br>
Mar 12 23:24:26 Sailfish unknown: type=1320 audit(1552425866.019:51):<br>
Mar 12 23:24:26 Sailfish kernel: audit: audit_lost=136 audit_rate_limit=20 audit_backlog_limit=64<br>
Mar 12 23:24:26 Sailfish kernel: audit: rate limit exceeded<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: library "libgui.so" wasn't loaded and RTLD_NOLOAD prevented it<br>
Mar 12 23:24:26 Sailfish systemd[3344]: Started Time Daemon.<br>
Mar 12 23:24:26 Sailfish DSME[2619]: IPHB: alarm state from timed: powerup=0, resume=0<br>
Mar 12 23:24:26 Sailfish kernel: msm_pm_qos_add_request: add request<br>
Mar 12 23:24:26 Sailfish kernel: msm_cci_init:1426: hw_version = 0x10020005<br>
Mar 12 23:24:26 Sailfish kernel: s5k5e8_ofilm probe succeeded<br>
Mar 12 23:24:26 Sailfish kernel: msm_cci_init:1426: hw_version = 0x10020005<br>
Mar 12 23:24:26 Sailfish kernel: s5k3l8_ofilm probe succeeded<br>
Mar 12 23:24:26 Sailfish kernel: msm_csid_init: CSID_VERSION = 0x30050001<br>
Mar 12 23:24:26 Sailfish kernel: msm_csid_irq CSID2_IRQ_STATUS_ADDR = 0x800<br>
Mar 12 23:24:26 Sailfish kernel: msm_csid_init: CSID_VERSION = 0x30050001<br>
Mar 12 23:24:26 Sailfish kernel: msm_csid_irq CSID0_IRQ_STATUS_ADDR = 0x800<br>
Mar 12 23:24:26 Sailfish kernel: pil-q6v5-mss 4080000.qcom,mss: modem: Brought out of reset<br>
Mar 12 23:24:26 Sailfish kernel: Doesn't support control clock.<br>
Mar 12 23:24:26 Sailfish kernel: This kernel doesn't support control clk in AP<br>
Mar 12 23:24:26 Sailfish droid-hal-init: Service 'qcom-sh' (pid 2927) exited with status 0<br>
Mar 12 23:24:26 Sailfish kernel: pil-q6v5-mss 4080000.qcom,mss: modem: Power/Clock ready interrupt received<br>
Mar 12 23:24:26 Sailfish kernel: pil-q6v5-mss 4080000.qcom,mss: Subsystem error monitoring/handling services are up<br>
Mar 12 23:24:26 Sailfish kernel: IPA received MPSS AFTER_POWERUP<br>
Mar 12 23:24:26 Sailfish kernel: IPA AFTER_POWERUP handling is complete<br>
Mar 12 23:24:26 Sailfish kernel: M-Notify: General: 7<br>
Mar 12 23:24:26 Sailfish kernel: IRQ has been disabled.<br>
Mar 12 23:24:26 Sailfish kernel: goodix_fp soc:goodix_fp: Selected 'goodixfp_reset_reset'<br>
Mar 12 23:24:26 Sailfish kernel: goodix_fp soc:goodix_fp: Selected 'goodixfp_reset_active'<br>
Mar 12 23:24:26 Sailfish kernel: goodix_fp soc:goodix_fp: IRQ after reset 0<br>
Mar 12 23:24:26 Sailfish kernel: QSEECOM: qseecom_unload_app: App id 5 now unloaded<br>
Mar 12 23:24:26 Sailfish kernel: gf:[info]  entergf_cleanup<br>
Mar 12 23:24:26 Sailfish kernel: gf:remove irq_gpio success<br>
Mar 12 23:24:26 Sailfish kernel: gf:remove reset_gpio success<br>
Mar 12 23:24:26 Sailfish kernel: gf:gx  fingerprint_pinctrl  release success<br>
Mar 12 23:24:26 Sailfish kernel: ---- power off ----<br>
Mar 12 23:24:26 Sailfish droid-hal-init: Service 'gx_fpd' is being killed...<br>
Mar 12 23:24:26 Sailfish droid-hal-init: Starting service 'fingerprintd'...<br>
Mar 12 23:24:26 Sailfish droid-hal-init: couldn't write 3701 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:26 Sailfish droid-hal-init: Service 'gx_fpd' (pid 2841) killed by signal 9<br>
Mar 12 23:24:26 Sailfish droid-hal-init: Service 'gx_fpd' (pid 2841) killing any children in process group<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: == hwcomposer module ==<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: * Address: 0xed7ce004<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: * Module API Version: 2<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: * HAL API Version: 0<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: * Identifier: hwcomposer<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: * Name: QTI Hardware Composer Module<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: * Author: CodeAurora Forum<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: == hwcomposer module ==<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: library "libsdmextension.so" not found<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: library "libsdmextension.so" not found<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: library "libsdm-color.so" not found<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: library "libsdmextension.so" not found<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: library "libsdmextension.so" not found<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: library "libsdm-diag.so" not found<br>
Mar 12 23:24:26 Sailfish kernel: [info] goodix_fb_state_chg_callback go to the goodix_fb_state_chg_callback value = 16<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: [D] unknown:0 - EGLFS: Screen Info<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: [D] unknown:0 -  - Physical size: QSizeF(69.0982, 122.226)<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: [D] unknown:0 -  - Screen size: QSize(1080, 1920)<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: [D] unknown:0 -  - Screen depth: 32<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: == hwcomposer device ==<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: * Version: 1050001 (interpreted as 1050001)<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: * Module: 0xed7ce004<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: == hwcomposer device ==<br>
Mar 12 23:24:26 Sailfish kernel: sysmon-qmi: sysmon_clnt_svc_arrive: Connection established between QMI handle and modem's SSCTL service<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: Could not stat() file '/proc/3706', Error when getting information for file ?/proc/3706?: No such<br>
 file or directoryCould not stat() file '/proc/3708', Error when getting information for file ?/proc/3708?: No such file or directoryFoun<br>
d 0 PIDs?<br>
Mar 12 23:24:26 Sailfish kernel: subsys-pil-tz soc:qcom,kgsl-hyp: a506_zap: loading from 0x000000008f000000 to 0x000000008f002000<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: Setting database locations<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: Checking database directories exist<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: Checking database version<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: Could not find database version file:'/home/nemo/.cache/tracker/db-version.txt'<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: Current databases are either old or no databases are set up yet<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: A reindex will be forced<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: Creating version file '/home/nemo/.cache/tracker/db-version.txt'<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: Checking database files exist<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: Removing all database/storage files<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: Removing database:'/home/nemo/.cache/tracker/meta.db'<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: Removing db-locale file:'/home/nemo/.cache/tracker/db-locale.txt'<br>
Mar 12 23:24:26 Sailfish kernel: MSM-CPP cpp_init_hardware:869 CPP HW Version: 0x40030003<br>
Mar 12 23:24:26 Sailfish kernel: MSM-CPP cpp_init_hardware:887 stream_cnt:0<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: oneshot: /etc/oneshot.d/100000/tracker-configs.sh - OK<br>
Mar 12 23:24:26 Sailfish kernel: subsys-pil-tz soc:qcom,kgsl-hyp: a506_zap: Brought out of reset<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: Not initializing tracker index on device<br>
Mar 12 23:24:26 Sailfish kernel: devfreq soc:qcom,kgsl-busmon: Couldn't update frequency transition information.<br>
Mar 12 23:24:26 Sailfish oneshot[3531]: oneshot: /etc/oneshot.d/100000/zz-initialize-tracker-index - OK<br>
Mar 12 23:24:26 Sailfish systemd[3344]: Started Oneshot stuff for user.<br>
Mar 12 23:24:26 Sailfish jolla-startupwizard-pre-user-session[3542]: [W] unknown:0 - QEglScreen 0x2321f0<br>
Mar 12 23:24:26 Sailfish kernel: apr_tal:Modem Is Up<br>
Mar 12 23:24:26 Sailfish mce[2275]: modules/proximity.c: report_proximity(): state: CLOSED - OPEN<br>
Mar 12 23:24:26 Sailfish kernel: diag: In diag_send_feature_mask_update, control channel is not open, p: 0, 0000000000000000<br>
Mar 12 23:24:26 Sailfish kernel: Sending QMI_IPA_INIT_MODEM_DRIVER_REQ_V01<br>
Mar 12 23:24:26 Sailfish kernel: ipa-wan handle_indication_req:133 not send indication<br>
Mar 12 23:24:26 Sailfish kernel: ipa ipa_uc_response_hdlr:461 IPA uC loaded<br>
Mar 12 23:24:26 Sailfish kernel: ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
Mar 12 23:24:26 Sailfish kernel: QMI_IPA_INIT_MODEM_DRIVER_REQ_V01 response received<br>
Mar 12 23:24:26 Sailfish kernel: ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
Mar 12 23:24:26 Sailfish kernel: ipa ipa_uc_wdi_event_log_info_handler:322 WDI feature missing 0x1<br>
Mar 12 23:24:26 Sailfish kernel: ipa ipa_uc_ntn_event_log_info_handler:39 NTN feature missing 0x1<br>
Mar 12 23:24:26 Sailfish kernel: ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
Mar 12 23:24:26 Sailfish kernel: msm_pm_qos_update_request: update request 100<br>
Mar 12 23:24:26 Sailfish kernel: msm_csid_init: CSID_VERSION = 0x30050001<br>
Mar 12 23:24:26 Sailfish kernel: msm_csid_irq CSID0_IRQ_STATUS_ADDR = 0x800<br>
Mar 12 23:24:26 Sailfish kernel: MSM-CPP cpp_init_hardware:869 CPP HW Version: 0x40030003<br>
Mar 12 23:24:26 Sailfish kernel: MSM-CPP cpp_init_hardware:887 stream_cnt:0<br>
Mar 12 23:24:26 Sailfish kernel: msm_cci_init:1426: hw_version = 0x10020005<br>
Mar 12 23:24:26 Sailfish wait-session-to-start[2852]: In wait loop (2) - state: active<br>
Mar 12 23:24:26 Sailfish wait-session-to-start[2852]: user@100000.service started<br>
Mar 12 23:24:26 Sailfish systemd[1]: Started Start User Session.<br>
Mar 12 23:24:26 Sailfish systemd[1]: Starting Inform dsme about runlevel change...<br>
Mar 12 23:24:26 Sailfish runlevel-change-done[3859]: Reached new runlevel USER<br>
Mar 12 23:24:26 Sailfish systemd[1]: Started Inform dsme about runlevel change.<br>
Mar 12 23:24:26 Sailfish rfkill[3868]: unblock set for all<br>
Mar 12 23:24:26 Sailfish kernel: msm_pm_qos_update_request: update request -1<br>
Mar 12 23:24:26 Sailfish kernel: hcismd_set_enable 1<br>
Mar 12 23:24:26 Sailfish kernel: Bluetooth: Cannot open the command channel<br>
Mar 12 23:24:27 Sailfish kernel: scm_call failed: func id 0x42000c16, ret: -1, syscall returns: 0x0, 0x0, 0x0<br>
Mar 12 23:24:27 Sailfish kernel: hyp_assign_table: Failed to assign memory protection, ret = -5<br>
Mar 12 23:24:27 Sailfish kernel: memshare: hyp_assign_phys failed size=2097152 err=-5<br>
Mar 12 23:24:27 Sailfish droid-hal-init: Starting service 'ims_rtp_daemon'...<br>
Mar 12 23:24:27 Sailfish droid-hal-init: Starting service 'imscmservice'...<br>
Mar 12 23:24:27 Sailfish droid-hal-init: couldn't write 3996 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:27 Sailfish droid-hal-init: couldn't write 3997 to /dev/cpuset/system-background/tasks: No space left on device<br>
Mar 12 23:24:27 Sailfish kernel: wcnss: NV download<br>
Mar 12 23:24:27 Sailfish kernel: wcnss: NV bin size: 31719, total_fragments: 11<br>
Mar 12 23:24:27 Sailfish kernel: wcnss: no space available for smd frame<br>
Mar 12 23:24:27 Sailfish kernel: wcnss: no space available for smd frame<br>
Mar 12 23:24:27 Sailfish kernel: wcnss: no space available for smd frame<br>
Mar 12 23:24:27 Sailfish kernel: wcnss: no space available for smd frame<br>
Mar 12 23:24:27 Sailfish rfkill[4126]: unblock set for all<br>
Mar 12 23:24:27 Sailfish kernel: hcismd_set_enable 1<br>
Mar 12 23:24:27 Sailfish kernel: Bluetooth: opening HCI-SMD channel :APPS_RIVA_BT_ACL<br>
Mar 12 23:24:27 Sailfish kernel: Bluetooth: opening HCI-SMD channel :APPS_RIVA_BT_CMD<br>
Mar 12 23:24:27 Sailfish kernel: Bluetooth: HCI device registration is starting<br>
Mar 12 23:24:27 Sailfish systemd[1]: Reached target Bluetooth.<br>
Mar 12 23:24:27 Sailfish kernel: msm_pm_qos_update_request: update request 100<br>
Mar 12 23:24:27 Sailfish kernel: msm_csid_init: CSID_VERSION = 0x30050001<br>
Mar 12 23:24:27 Sailfish kernel: msm_csid_irq CSID2_IRQ_STATUS_ADDR = 0x800<br>
Mar 12 23:24:27 Sailfish kernel: MSM-CPP cpp_init_hardware:869 CPP HW Version: 0x40030003<br>
Mar 12 23:24:27 Sailfish kernel: MSM-CPP cpp_init_hardware:887 stream_cnt:0<br>
Mar 12 23:24:27 Sailfish ofonod[2637]: RIL version 13<br>
Mar 12 23:24:27 Sailfish kernel: msm_pm_qos_update_request: update request -1<br>
Mar 12 23:24:27 Sailfish kernel: msm_cci_init:1426: hw_version = 0x10020005<br>
Mar 12 23:24:27 Sailfish kernel: ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
Mar 12 23:24:27 Sailfish kernel: ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
Mar 12 23:24:27 Sailfish kernel: ipa-wan ipa_wwan_ioctl:1542 get AGG size 8192 count 10<br>
Mar 12 23:24:27 Sailfish kernel: ipa ipa_sps_irq_control_all:938 EP (5) not allocated.<br>
Mar 12 23:24:27 Sailfish kernel: ipa ipa_assign_policy:3112 get close-by 8192<br>
Mar 12 23:24:27 Sailfish kernel: ipa ipa_assign_policy:3118 set rx_buff_sz 7808<br>
Mar 12 23:24:27 Sailfish kernel: ipa ipa_assign_policy:3141 set aggr_limit 6<br>
Mar 12 23:24:27 Sailfish kernel: ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data0) register to IPA<br>
Mar 12 23:24:27 Sailfish audit: NETFILTER_CFG table=mangle family=2 entries=20<br>
Mar 12 23:24:27 Sailfish audit[4206]: SYSCALL arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40 a3=7f8d839e00 items=0 ppid=284<br>
7 pid=4206 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system<br>
/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:27 Sailfish audit: AUDIT1327 proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D74006D616E676C65002D4100505245524F5<br>
554494E47002D6D00736F636B6574002D2D6E6F77696C6463617264002D2D726573746F72652D736B6D61726B002D6A00414343455054<br>
Mar 12 23:24:27 Sailfish unknown: type=1325 audit(1552425867.991:55): table=mangle family=2 entries=20<br>
Mar 12 23:24:27 Sailfish unknown: type=1300 audit(1552425867.991:55): arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40<br>
a3=7f8d839e00 items=0 ppid=2847 pid=4206 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=429496729<br>
5 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:28 Sailfish unknown: type=1327 audit(1552425867.991:55): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D7<br>
4006D616E676C65002D4100505245524F5554494E47002D6D00736F636B6574002D2D6E6F77696C6463617264002D2D726573746F72652D736B6D61726B002D6A00414343<br>
455054<br>
Mar 12 23:24:28 Sailfish unknown: type=1320 audit(1552425867.991:55):<br>
Mar 12 23:24:28 Sailfish droid-hal-init: property_set("ro.ril.svlte1x", "false") failed<br>
Mar 12 23:24:28 Sailfish droid-hal-init: property_set("ro.ril.svdo", "false") failed<br>
Mar 12 23:24:28 Sailfish audit: NETFILTER_CFG table=mangle family=2 entries=21<br>
Mar 12 23:24:28 Sailfish audit[4207]: SYSCALL arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40 a3=7fb5c39e00 items=0 ppid=284<br>
7 pid=4207 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system<br>
/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:28 Sailfish audit: AUDIT1327 proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D74006D616E676C65002D4400505245524F5<br>
554494E47002D6D00736F636B6574002D2D6E6F77696C6463617264002D2D726573746F72652D736B6D61726B002D6A00414343455054<br>
Mar 12 23:24:28 Sailfish unknown: type=1325 audit(1552425868.003:56): table=mangle family=2 entries=21<br>
Mar 12 23:24:28 Sailfish unknown: type=1300 audit(1552425868.003:56): arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40<br>
a3=7fb5c39e00 items=0 ppid=2847 pid=4207 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=429496729<br>
5 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:28 Sailfish unknown: type=1327 audit(1552425868.003:56): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D7<br>
4006D616E676C65002D4400505245524F5554494E47002D6D00736F636B6574002D2D6E6F77696C6463617264002D2D726573746F72652D736B6D61726B002D6A00414343<br>
455054<br>
Mar 12 23:24:28 Sailfish unknown: type=1320 audit(1552425868.003:56):<br>
Mar 12 23:24:28 Sailfish kernel: IPC_RTR: msm_ipc_router_bind: slim_daemon Do not have permissions<br>
Mar 12 23:24:28 Sailfish audit: NETFILTER_CFG table=raw family=2 entries=3<br>
Mar 12 23:24:28 Sailfish audit[4212]: SYSCALL arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40 a3=7fa7640000 items=0 ppid=284<br>
7 pid=4212 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system<br>
/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:28 Sailfish kernel: ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data1) register to IPA<br>
Mar 12 23:24:28 Sailfish unknown: type=1325 audit(1552425868.015:57): table=raw family=2 entries=3<br>
Mar 12 23:24:28 Sailfish unknown: type=1300 audit(1552425868.015:57): arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40<br>
a3=7fa7640000 items=0 ppid=2847 pid=4212 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=429496729<br>
5 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:28 Sailfish audit: AUDIT1327 proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D7400726177002D4E006E6D5F6D646D70727<br>
8795F7261775F707265<br>
Mar 12 23:24:28 Sailfish unknown: type=1327 audit(1552425868.015:57): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D7<br>
400726177002D4E006E6D5F6D646D707278795F7261775F707265<br>
Mar 12 23:24:28 Sailfish unknown: type=1320 audit(1552425868.015:57):<br>
Mar 12 23:24:28 Sailfish audit: NETFILTER_CFG table=mangle family=2 entries=20<br>
Mar 12 23:24:28 Sailfish audit[4226]: SYSCALL arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40 a3=7f7961d000 items=0 ppid=284<br>
7 pid=4226 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system<br>
/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:28 Sailfish audit: AUDIT1327 proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D74006D616E676C65002D4E006E6D5F6D646<br>
D707278795F6D6E676C5F706F7374<br>
Mar 12 23:24:28 Sailfish unknown: type=1325 audit(1552425868.027:58): table=mangle family=2 entries=20<br>
Mar 12 23:24:28 Sailfish unknown: type=1300 audit(1552425868.027:58): arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40<br>
a3=7f7961d000 items=0 ppid=2847 pid=4226 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=429496729<br>
5 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:28 Sailfish unknown: type=1327 audit(1552425868.027:58): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D7<br>
4006D616E676C65002D4E006E6D5F6D646D707278795F6D6E676C5F706F7374<br>
Mar 12 23:24:28 Sailfish unknown: type=1320 audit(1552425868.027:58):<br>
Mar 12 23:24:28 Sailfish audit: NETFILTER_CFG table=mangle family=2 entries=22<br>
Mar 12 23:24:28 Sailfish audit[4228]: SYSCALL arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40 a3=7f8301d000 items=0 ppid=284<br>
7 pid=4228 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="iptables" exe="/system<br>
/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:28 Sailfish audit: AUDIT1327 proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D74006D616E676C65002D4E006E6D5F6D646<br>
D707278795F6D6E676C5F7072655F737069<br>
Mar 12 23:24:28 Sailfish unknown: type=1325 audit(1552425868.039:59): table=mangle family=2 entries=22<br>
Mar 12 23:24:28 Sailfish unknown: type=1300 audit(1552425868.039:59): arch=c00000b7 syscall=208 success=yes exit=0 a0=c a1=0 a2=40<br>
a3=7f8301d000 items=0 ppid=2847 pid=4228 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=429496729<br>
5 comm="iptables" exe="/system/bin/iptables" subj=kernel key=(null)<br>
Mar 12 23:24:28 Sailfish unknown: type=1327 audit(1552425868.039:59): proctitle=2F73797374656D2F62696E2F69707461626C6573002D77002D7<br>
4006D616E676C65002D4E006E6D5F6D646D707278795F6D6E676C5F7072655F737069<br>
Mar 12 23:24:28 Sailfish unknown: type=1320 audit(1552425868.039:59):<br>
Mar 12 23:24:28 Sailfish kernel: audit: audit_lost=148 audit_rate_limit=20 audit_backlog_limit=64<br>
Mar 12 23:24:28 Sailfish kernel: audit: rate limit exceeded<br>
Mar 12 23:24:28 Sailfish ofonod[2637]: RIL version 13<br>
Mar 12 23:24:28 Sailfish kernel: ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data2) register to IPA<br>
Mar 12 23:24:28 Sailfish kernel: ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data3) register to IPA<br>
Mar 12 23:24:28 Sailfish kernel: ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data4) register to IPA<br>
Mar 12 23:24:28 Sailfish kernel: ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data5) register to IPA<br>
Mar 12 23:24:28 Sailfish kernel: ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data6) register to IPA<br>
Mar 12 23:24:28 Sailfish kernel: ipa-wan ipa_wwan_ioctl:1443 dev(rmnet_data7) register to IPA<br>
Mar 12 23:24:28 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
Mar 12 23:24:28 Sailfish mce[2275]: tklock.c: tklock_datapipe_uiexception_type_cb(): uiexception_type = none - notif<br>
Mar 12 23:24:28 Sailfish jolla-startupwizard-pre-user-session[3542]: [W] unknown:0 - StartupWizardManager: cannot load translation<br>
file "startupwizard" for locale ""<br>
Mar 12 23:24:28 Sailfish jolla-startupwizard-pre-user-session[3542]: [D] _createMdmDialog:102 - Not loading MDM terms: file:///usr/share/<br>
jolla-startupwizard-pre-user-session/MdmTermsOfUseDialog.qml:4 module "Sailfish.Mdm" is not installed<br>
Mar 12 23:24:28 Sailfish jolla-startupwizard-pre-user-session[3542]: [W] unknown:738 - file:///usr/lib/qt5/qml/Sailfish/Silica/Page<br>
Stack.qml:738:17: Unable to assign [undefined] to QObject*<br>
Mar 12 23:24:28 Sailfish mce[2275]: modules/display.c: mdy_stm_set_compositor_availability_changed(): compositor availability chang<br>
e: pending<br>
Mar 12 23:24:28 Sailfish mce[2275]: modules/display.c: mdy_stm_step(): forced brightness sync to: 81<br>
Mar 12 23:24:28 Sailfish mce[2275]: modules/display.c: mdy_stm_set_compositor_availability_changed(): compositor availability chang<br>
e: handled<br>
Mar 12 23:24:28 Sailfish dbus-daemon[3547]: Activating via systemd: service name='org.maliit.server' unit='maliit-server.service'<br>
Mar 12 23:24:28 Sailfish jolla-startupwizard-pre-user-session[3542]: [D] unknown:0 - unsleepDisplay<br>
Mar 12 23:24:28 Sailfish jolla-startupwizard-pre-user-session[3542]: [W] unknown:0 - QEglWindow 0x269000: 0x2583a8 0x0<br>
Mar 12 23:24:28 Sailfish systemd[3344]: Starting Runs basic setup for the Jolla user before the rest of the user session is started...<br>
Mar 12 23:24:28 Sailfish rfkill[4392]: unblock set for all<br>
Mar 12 23:24:28 Sailfish rfkill[4393]: unblock set for all<br>
Mar 12 23:24:28 Sailfish sh[2583]: /usr/bin/droid/droid-hcismd-up.sh: line 19: hciconfig: command not found<br>
Mar 12 23:24:28 Sailfish systemd[3344]: Started Runs basic setup for the Jolla user before the rest of the user session is started.<br>
Mar 12 23:24:28 Sailfish kernel: hcismd_set_enable 1<br>
Mar 12 23:24:28 Sailfish kernel: hcismd_set_enable 1<br>
Mar 12 23:24:28 Sailfish kernel: Bluetooth: HCI device un-registration going on<br>
Mar 12 23:24:28 Sailfish kernel: Bluetooth: Frame for unknown HCI device (hdev=NULL)<br>
Mar 12 23:24:28 Sailfish kernel: Bluetooth: Frame for unknown HCI device (hdev=NULL)<br>
Mar 12 23:24:28 Sailfish sh[2583]: grep: invalid option -- 'P'<br>
Mar 12 23:24:28 Sailfish sh[2583]: BusyBox v1.29.3 (2019-03-03 16:24:36 EST) multi-call binary.<br>
Mar 12 23:24:28 Sailfish sh[2583]: Usage: grep [-HhnlLoqvsriwFEz] [-m N] [-A/B/C N] PATTERN/-e PATTERN.../-f FILE [FILE]...<br>
Mar 12 23:24:28 Sailfish sh[2583]: Search for PATTERN in FILEs (or stdin)<br>
Mar 12 23:24:28 Sailfish sh[2583]: -H        Add 'filename:' prefix<br>
Mar 12 23:24:28 Sailfish sh[2583]: -h        Do not add 'filename:' prefix<br>
Mar 12 23:24:28 Sailfish sh[2583]: -n        Add 'line_no:' prefix<br>
Mar 12 23:24:28 Sailfish sh[2583]: -l        Show only names of files that match<br>
Mar 12 23:24:28 Sailfish sh[2583]: -L        Show only names of files that don't match<br>
Mar 12 23:24:28 Sailfish sh[2583]: -c        Show only count of matching lines<br>
Mar 12 23:24:28 Sailfish sh[2583]: -o        Show only the matching part of line<br>
Mar 12 23:24:28 Sailfish sh[2583]: -q        Quiet. Return 0 if PATTERN is found, 1 otherwise<br>
Mar 12 23:24:28 Sailfish sh[2583]: -v        Select non-matching lines<br>
Mar 12 23:24:28 Sailfish sh[2583]: -s        Suppress open and read errors<br>
Mar 12 23:24:28 Sailfish sh[2583]: -r        Recurse<br>
Mar 12 23:24:28 Sailfish sh[2583]: -i        Ignore case<br>
Mar 12 23:24:28 Sailfish sh[2583]: -w        Match whole words only<br>
Mar 12 23:24:28 Sailfish sh[2583]: -x        Match whole lines only<br>
Mar 12 23:24:28 Sailfish sh[2583]: -F        PATTERN is a literal (not regexp)<br>
Mar 12 23:24:28 Sailfish sh[2583]: -E        PATTERN is an extended regexp<br>
Mar 12 23:24:28 Sailfish sh[2583]: -z        Input is NUL terminated<br>
Mar 12 23:24:28 Sailfish sh[2583]: -m N        Match up to N times per file<br>
Mar 12 23:24:28 Sailfish sh[2583]: -A N        Print N lines of trailing context<br>
Mar 12 23:24:28 Sailfish sh[2583]: -B N        Print N lines of leading context<br>
Mar 12 23:24:28 Sailfish sh[2583]: -C N        Same as '-A N -B N'<br>
Mar 12 23:24:28 Sailfish sh[2583]: -e PTRN        Pattern to match<br>
Mar 12 23:24:28 Sailfish sh[2583]: -f FILE        Read pattern from file<br>
Mar 12 23:24:28 Sailfish systemd[1]: bluetooth.target: Unit not needed anymore. Stopping.<br>
Mar 12 23:24:28 Sailfish systemd[1]: Stopped target Bluetooth.<br>
Mar 12 23:24:28 Sailfish mce[2275]: modules/display.c: mdy_topmost_window_pid_reply_cb(): error reply: org.freedesktop.DBus.Error.U<br>
nknownMethod: No such method 'privateTopmostWindowProcessId' in interface 'org.nemomobile.compositor' at object path '/' (signature '')<br>
<br>
Mar 12 23:24:28 Sailfish mce[2275]: modules/display.c: mdy_display_state_enter(): current display state = ON<br>
Mar 12 23:24:28 Sailfish sh[2583]: BT MAC:<br>
Mar 12 23:24:28 Sailfish kernel: hcismd_set_enable 0<br>
Mar 12 23:24:28 Sailfish kernel: Bluetooth: opening HCI-SMD channel :APPS_RIVA_BT_ACL<br>
Mar 12 23:24:28 Sailfish kernel: Bluetooth: opening HCI-SMD channel :APPS_RIVA_BT_CMD<br>
Mar 12 23:24:28 Sailfish kernel: Bluetooth: HCI device registration is starting<br>
Mar 12 23:24:28 Sailfish systemd[1]: Started Enable Bluetooth HCI over SMD.<br>
Mar 12 23:24:28 Sailfish systemd[1]: Starting Enable FM Radio...<br>
Mar 12 23:24:28 Sailfish systemd[1]: Started Enable FM Radio.<br>
Mar 12 23:24:28 Sailfish systemd[1]: Reached target Bluetooth.<br>
Mar 12 23:24:29 Sailfish kernel: HTB: quantum of class 10001 is big. Consider r2q change.<br>
Mar 12 23:24:29 Sailfish kernel: HTB: quantum of class 10010 is big. Consider r2q change.<br>
Mar 12 23:24:29 Sailfish kernel: NOHZ: local_softirq_pending 08<br>
Mar 12 23:24:30 Sailfish kernel: NOHZ: local_softirq_pending 08<br>
Mar 12 23:24:31 Sailfish kernel: NOHZ: local_softirq_pending 08<br>
Mar 12 23:24:32 Sailfish kernel: NOHZ: local_softirq_pending 08<br>
Mar 12 23:24:37 Sailfish kernel: wlan: WCNSS WLAN version 1.5.1.2<br>
Mar 12 23:24:37 Sailfish kernel: wlan: WCNSS software version CNSS-PR-4-0-00328<br>
Mar 12 23:24:37 Sailfish kernel: wlan: WCNSS hardware version WCN v2.0 RadioPhy vIris_TSMC_4.0 with 48MHz XO<br>
Mar 12 23:24:37 Sailfish kernel: DefaultCountry is 00<br>
Mar 12 23:24:37 Sailfish kernel: wlan_logging_sock_activate_svc: Initalizing FEConsoleLog = 1 NumBuff = 32<br>
Mar 12 23:24:37 Sailfish kernel: wlan_logging_sock_activate_svc: Initalizing Pkt stats pkt_stats_buff = 16<br>
Mar 12 23:24:37 Sailfish kernel: send_filled_buffers_to_user: Send Failed -3 drop_count = 0<br>
Mar 12 23:24:37 Sailfish kernel: wlan: driver loaded<br>
Mar 12 23:24:37 Sailfish kernel: SMBCHG: wait_for_src_detect: src detect didnt go to a lowered state, still at high, tries = 2, rc<br>
= 0<br>
Mar 12 23:24:37 Sailfish kernel: SMBCHG: rerun_apsd: wait for src detect failed rc = -22<br>
Mar 12 23:24:37 Sailfish kernel: msm-dwc3 7000000.ssusb: DWC3 exited from low power mode<br>
Mar 12 23:24:37 Sailfish mce[2275]: modules/battery-udev.c: mcebat_update(): charger_state: off - on<br>
Mar 12 23:24:37 Sailfish kernel: enable_store: android_usb: already disabled<br>
Mar 12 23:24:37 Sailfish kernel: rndis_function_bind_config: rndis_function_bind_config MAC: E6:EE:29:A5:FA:BA<br>
Mar 12 23:24:37 Sailfish kernel: android_usb gadget: using random self ethernet address<br>
Mar 12 23:24:37 Sailfish kernel: android_usb gadget: using previous host ethernet address<br>
Mar 12 23:24:37 Sailfish kernel: rndis0: MAC 06:c1:22:62:de:3a<br>
Mar 12 23:24:37 Sailfish kernel: rndis0: HOST MAC ee:60:50:97:8f:8c<br>
Mar 12 23:24:37 Sailfish kernel: hid keyboard<br>
Mar 12 23:24:37 Sailfish kernel: hidg_bind: creating device ffffffc0840d1200<br>
Mar 12 23:24:37 Sailfish kernel: hid mouse<br>
Mar 12 23:24:37 Sailfish kernel: hidg_bind: creating device ffffffc0840d0a00<br>
Mar 12 23:24:37 Sailfish kernel: IPv6: ADDRCONF(NETDEV_UP): rndis0: link is not ready<br>
Mar 12 23:24:37 Sailfish usb_moded[2592]: EXEC ifconfig rndis0 192.168.2.15 255.255.255.0<br>
                                          ; exit code is 256<br>
Mar 12 23:24:37 Sailfish kernel: IPv6: ADDRCONF(NETDEV_UP): rndis0: link is not ready<br>
Mar 12 23:24:37 Sailfish kernel: [21:24:37.606528] [0000000029510524] [VosMC]  wlan: [E :HDD] Ptt Socket error sending message to t<br>
he app!!<br>
Mar 12 23:24:37 Sailfish kernel: [21:24:37.606651] [0000000029510E51] [VosMC]  wlan: [E :HDD] Ptt Socket error sending message to t<br>
he app!!<br>
Mar 12 23:24:37 Sailfish kernel: [21:24:37.607666] [0000000029515A6F] [VosMC]  wlan: [E :HDD] Ptt Socket error sending message to t<br>
he app!!<br>
Mar 12 23:24:37 Sailfish kernel: [21:24:37.806518] [00000000298B9C54] [VosMC]  wlan: [E :HDD] Ptt Socket error sending message to t<br>
he app!!<br>
Mar 12 23:24:37 Sailfish kernel: android_work: android_work: sent uevent USB_STATE=CONNECTED<br>
Mar 12 23:24:37 Sailfish kernel: android_work: android_work: sent uevent USB_STATE=DISCONNECTED<br>
Mar 12 23:24:37 Sailfish systemd[1]: Started udhcpcd DHCP server.<br>
Mar 12 23:24:37 Sailfish udhcpd[4562]: udhcpd: started, v1.29.3<br>
Mar 12 23:24:37 Sailfish udhcpd[4562]: udhcpd: can't open '/var/lib/misc/udhcpd.leases': No such file or directory<br>
Mar 12 23:24:38 Sailfish kernel: android_work: android_work: sent uevent USB_STATE=CONNECTED<br>
Mar 12 23:24:38 Sailfish kernel: android_usb gadget: high-speed config #1: 86000c8.android_usb<br>
Mar 12 23:24:38 Sailfish kernel: msm-dwc3 7000000.ssusb: Avail curr from USB = 500<br>
Mar 12 23:24:38 Sailfish kernel: IPv6: ADDRCONF(NETDEV_CHANGE): rndis0: link becomes ready<br>
Mar 12 23:24:38 Sailfish kernel: android_work: android_work: sent uevent USB_STATE=CONFIGURED<br>
Mar 12 23:24:38 Sailfish kernel: tz_get_target_freq ADRENO jumping level = 5 last_level = 6 total=12304 busy=12335 original busy_time=123<br>
35<br>
Mar 12 23:24:38 Sailfish jolla-startupwizard-pre-user-session[3542]: [W] unknown:206 - file:///usr/lib/qt5/qml/Sailfish/Silica/Dial<br>
ogHeader.qml:206:13: QML Binding: Property 'forwardIndicatorHighlighted' does not exist on QtObject.<br>
Mar 12 23:24:38 Sailfish jolla-startupwizard-pre-user-session[3542]: [W] unknown:153 - file:///usr/lib/qt5/qml/Sailfish/Silica/Dial<br>
ogHeader.qml:153:13: QML Binding: Property 'backIndicatorHighlighted' does not exist on QtObject.<br>
Mar 12 23:24:39 Sailfish kernel: tz_get_target_freq ADRENO jumping level = 6 last_level = 5 total=27313 busy=6314 original busy_time=6314<br>
<br>
Mar 12 23:24:39 Sailfish kernel: tz_get_target_freq ADRENO jumping level = 5 last_level = 6 total=16484 busy=10116 original busy_time=101<br>
16<br>
Mar 12 23:24:39 Sailfish jolla-startupwizard-pre-user-session[3542]: [W] unknown:206 - file:///usr/lib/qt5/qml/Sailfish/Silica/Dial<br>
ogHeader.qml:206:13: QML Binding: Property 'forwardIndicatorHighlighted' does not exist on QtObject.<br>
Mar 12 23:24:39 Sailfish jolla-startupwizard-pre-user-session[3542]: [W] unknown:153 - file:///usr/lib/qt5/qml/Sailfish/Silica/Dial<br>
ogHeader.qml:153:13: QML Binding: Property 'backIndicatorHighlighted' does not exist on QtObject.<br>
Mar 12 23:24:40 Sailfish udhcpd[4562]: udhcpd: sending OFFER of 192.168.2.1<br>
Mar 12 23:24:40 Sailfish udhcpd[4562]: udhcpd: sending ACK to 192.168.2.1<br>
Mar 12 23:24:48 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
Mar 12 23:24:55 Sailfish kernel: [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister sit0<br>
Mar 12 23:24:55 Sailfish kernel: [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister sit0<br>
Mar 12 23:24:55 Sailfish kernel: [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister lo<br>
Mar 12 23:24:55 Sailfish kernel: [RMNET:HI] rmnet_config_notify_cb(): Kernel is trying to unregister lo<br>
Mar 12 23:24:59 Sailfish usb_moded[2592]: unexpected change '/sys/class/android_usb/android0/functions' : 'rndis' - 'rndis,hid'<br>
Mar 12 23:25:08 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
Mar 12 23:25:28 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
Mar 12 23:25:48 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
Mar 12 23:26:08 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
Mar 12 23:26:28 Sailfish dbus-daemon[3547]: Failed to activate service 'org.maliit.server': timed out<br>
Mar 12 23:26:28 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
Mar 12 23:26:32 Sailfish dbus-daemon[3547]: Activating via systemd: service name='org.maliit.server' unit='maliit-server.service'<br>
Mar 12 23:26:32 Sailfish systemd[3344]: Starting Runs basic setup for the Jolla user before the rest of the user session is started...<br>
Mar 12 23:26:32 Sailfish systemd[3344]: Started Runs basic setup for the Jolla user before the rest of the user session is started.<br>
Mar 12 23:26:48 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
Mar 12 23:27:08 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
Mar 12 23:27:28 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
Mar 12 23:27:48 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
Mar 12 23:28:08 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
Mar 12 23:28:28 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
Mar 12 23:28:32 Sailfish dbus-daemon[3547]: Failed to activate service 'org.maliit.server': timed out<br>
Mar 12 23:28:36 Sailfish dbus-daemon[3547]: Activating via systemd: service name='org.maliit.server' unit='maliit-server.service'<br>
Mar 12 23:28:36 Sailfish systemd[3344]: Starting Runs basic setup for the Jolla user before the rest of the user session is started...<br>
Mar 12 23:28:36 Sailfish systemd[3344]: Started Runs basic setup for the Jolla user before the rest of the user session is started.<br>
Mar 12 23:28:48 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
Mar 12 23:29:08 Sailfish mce[2275]: tklock.c: tklock_dbus_notification_beg_cb(): notification begin from name=:1.37 owner=:1.37 pid<br>
=3542 uid=100000 gid=100000 priv=0 cmd=/usr/bin/jolla-startupwizard-pre-user-session -plugin evdevtou<br>
</details><br>

Лог systemd рабочей Sailfish 3.0.2.8
```console
sh-3.2# systemctl
```
<details>
UNIT                                                LOAD   ACTIVE     SUB       JOB   DESCRIPTION<br>
sys-devices-soc-7000000.ssusb-7000000.dwc3-gadget-net-rndis0.device loaded active     plugged         /sys/devices/soc/7000000.ssusb/7000<br>
000.dwc3/gadget/<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p1.device loaded active     plugged         /sys/devices/soc/7<br>
824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p10.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p11.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p12.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p13.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p14.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p15.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p16.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p17.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p18.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p19.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p2.device loaded active     plugged         /sys/devices/soc/7<br>
824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p20.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p21.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p22.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p23.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p24.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p25.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p26.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p27.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p28.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p29.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p3.device loaded active     plugged         /sys/devices/soc/7<br>
824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p30.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p31.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p32.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p33.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p34.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p35.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p36.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p37.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p38.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p39.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p4.device loaded active     plugged         /sys/devices/soc/7<br>
824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p40.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p41.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p42.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p43.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p44.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p45.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p46.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p47.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p48.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p49.device loaded active     plugged         /sys/devices/soc/<br>
7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p5.device loaded active     plugged         /sys/devices/soc/7<br>
824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p6.device loaded active     plugged         /sys/devices/soc/7<br>
824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p7.device loaded active     plugged         /sys/devices/soc/7<br>
824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p8.device loaded active     plugged         /sys/devices/soc/7<br>
824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0p9.device loaded active     plugged         /sys/devices/soc/7<br>
824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0-mmcblk0rpmb.device loaded active     plugged         /sys/devices/soc<br>
/7824900.sdhci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-7824900.sdhci-mmc_host-mmc0-mmc0:0001-block-mmcblk0.device loaded active     plugged         /sys/devices/soc/7824900.sdh<br>
ci/mmc_host/mmc0/mmc0:0<br>
sys-devices-soc-a000000.qcom\x2cwcnss\x2dwlan-ieee80211-phy0-rfkill2.device loaded active     plugged         /sys/devices/soc/a000000.qc<br>
om,wcnss-wlan/ieee80211/<br>
sys-devices-soc-a000000.qcom\x2cwcnss\x2dwlan-net-p2p0.device loaded active     plugged         /sys/devices/soc/a000000.qcom,wcnss-wlan/<br>
net/p2p0<br>
sys-devices-soc-a000000.qcom\x2cwcnss\x2dwlan-net-wlan0.device loaded active     plugged         /sys/devices/soc/a000000.qcom,wcnss-wlan<br>
/net/wlan0<br>
sys-devices-soc-c051000.sound-sound-card0.device    loaded active     plugged         /sys/devices/soc/c051000.sound/sound/card0<br>
sys-devices-virtual-block-ram0.device               loaded active     plugged         /sys/devices/virtual/block/ram0<br>
sys-devices-virtual-block-ram1.device               loaded active     plugged         /sys/devices/virtual/block/ram1<br>
sys-devices-virtual-block-ram10.device              loaded active     plugged         /sys/devices/virtual/block/ram10<br>
sys-devices-virtual-block-ram11.device              loaded active     plugged         /sys/devices/virtual/block/ram11<br>
sys-devices-virtual-block-ram12.device              loaded active     plugged         /sys/devices/virtual/block/ram12<br>
sys-devices-virtual-block-ram13.device              loaded active     plugged         /sys/devices/virtual/block/ram13<br>
sys-devices-virtual-block-ram14.device              loaded active     plugged         /sys/devices/virtual/block/ram14<br>
sys-devices-virtual-block-ram15.device              loaded active     plugged         /sys/devices/virtual/block/ram15<br>
sys-devices-virtual-block-ram2.device               loaded active     plugged         /sys/devices/virtual/block/ram2<br>
sys-devices-virtual-block-ram3.device               loaded active     plugged         /sys/devices/virtual/block/ram3<br>
sys-devices-virtual-block-ram4.device               loaded active     plugged         /sys/devices/virtual/block/ram4<br>
sys-devices-virtual-block-ram5.device               loaded active     plugged         /sys/devices/virtual/block/ram5<br>
sys-devices-virtual-block-ram6.device               loaded active     plugged         /sys/devices/virtual/block/ram6<br>
sys-devices-virtual-block-ram7.device               loaded active     plugged         /sys/devices/virtual/block/ram7<br>
sys-devices-virtual-block-ram8.device               loaded active     plugged         /sys/devices/virtual/block/ram8<br>
sys-devices-virtual-block-ram9.device               loaded active     plugged         /sys/devices/virtual/block/ram9<br>
sys-devices-virtual-block-zram0.device              loaded active     plugged         /sys/devices/virtual/block/zram0<br>
sys-devices-virtual-bluetooth-hci0.device           loaded active     plugged         /sys/devices/virtual/bluetooth/hci0<br>
sys-devices-virtual-net-r_rmnet_data0.device        loaded active     plugged         /sys/devices/virtual/net/r_rmnet_data0<br>
sys-devices-virtual-net-r_rmnet_data1.device        loaded active     plugged         /sys/devices/virtual/net/r_rmnet_data1<br>
sys-devices-virtual-net-r_rmnet_data2.device        loaded active     plugged         /sys/devices/virtual/net/r_rmnet_data2<br>
sys-devices-virtual-net-r_rmnet_data3.device        loaded active     plugged         /sys/devices/virtual/net/r_rmnet_data3<br>
sys-devices-virtual-net-r_rmnet_data4.device        loaded active     plugged         /sys/devices/virtual/net/r_rmnet_data4<br>
sys-devices-virtual-net-r_rmnet_data5.device        loaded active     plugged         /sys/devices/virtual/net/r_rmnet_data5<br>
sys-devices-virtual-net-r_rmnet_data6.device        loaded active     plugged         /sys/devices/virtual/net/r_rmnet_data6<br>
sys-devices-virtual-net-r_rmnet_data7.device        loaded active     plugged         /sys/devices/virtual/net/r_rmnet_data7<br>
sys-devices-virtual-net-r_rmnet_data8.device        loaded active     plugged         /sys/devices/virtual/net/r_rmnet_data8<br>
sys-devices-virtual-net-rmnet_data0.device          loaded active     plugged         /sys/devices/virtual/net/rmnet_data0<br>
sys-devices-virtual-net-rmnet_data1.device          loaded active     plugged         /sys/devices/virtual/net/rmnet_data1<br>
sys-devices-virtual-net-rmnet_data2.device          loaded active     plugged         /sys/devices/virtual/net/rmnet_data2<br>
sys-devices-virtual-net-rmnet_data3.device          loaded active     plugged         /sys/devices/virtual/net/rmnet_data3<br>
sys-devices-virtual-net-rmnet_data4.device          loaded active     plugged         /sys/devices/virtual/net/rmnet_data4<br>
sys-devices-virtual-net-rmnet_data5.device          loaded active     plugged         /sys/devices/virtual/net/rmnet_data5<br>
sys-devices-virtual-net-rmnet_data6.device          loaded active     plugged         /sys/devices/virtual/net/rmnet_data6<br>
sys-devices-virtual-net-rmnet_data7.device          loaded active     plugged         /sys/devices/virtual/net/rmnet_data7<br>
sys-devices-virtual-net-rmnet_ipa0.device           loaded active     plugged         /sys/devices/virtual/net/rmnet_ipa0<br>
sys-devices-virtual-net-sit0.device                 loaded active     plugged         /sys/devices/virtual/net/sit0<br>
sys-module-configfs.device                          loaded active     plugged         /sys/module/configfs<br>
sys-module-fuse.device                              loaded active     plugged         /sys/module/fuse<br>
sys-subsystem-bluetooth-devices-hci0.device         loaded active     plugged         /sys/subsystem/bluetooth/devices/hci0<br>
sys-subsystem-net-devices-p2p0.device               loaded active     plugged         /sys/subsystem/net/devices/p2p0<br>
sys-subsystem-net-devices-r_rmnet_data0.device      loaded active     plugged         /sys/subsystem/net/devices/r_rmnet_data0<br>
sys-subsystem-net-devices-r_rmnet_data1.device      loaded active     plugged         /sys/subsystem/net/devices/r_rmnet_data1<br>
sys-subsystem-net-devices-r_rmnet_data2.device      loaded active     plugged         /sys/subsystem/net/devices/r_rmnet_data2<br>
sys-subsystem-net-devices-r_rmnet_data3.device      loaded active     plugged         /sys/subsystem/net/devices/r_rmnet_data3<br>
sys-subsystem-net-devices-r_rmnet_data4.device      loaded active     plugged         /sys/subsystem/net/devices/r_rmnet_data4<br>
sys-subsystem-net-devices-r_rmnet_data5.device      loaded active     plugged         /sys/subsystem/net/devices/r_rmnet_data5<br>
sys-subsystem-net-devices-r_rmnet_data6.device      loaded active     plugged         /sys/subsystem/net/devices/r_rmnet_data6<br>
sys-subsystem-net-devices-r_rmnet_data7.device      loaded active     plugged         /sys/subsystem/net/devices/r_rmnet_data7<br>
sys-subsystem-net-devices-r_rmnet_data8.device      loaded active     plugged         /sys/subsystem/net/devices/r_rmnet_data8<br>
sys-subsystem-net-devices-rmnet_data0.device        loaded active     plugged         /sys/subsystem/net/devices/rmnet_data0<br>
sys-subsystem-net-devices-rmnet_data1.device        loaded active     plugged         /sys/subsystem/net/devices/rmnet_data1<br>
sys-subsystem-net-devices-rmnet_data2.device        loaded active     plugged         /sys/subsystem/net/devices/rmnet_data2<br>
sys-subsystem-net-devices-rmnet_data3.device        loaded active     plugged         /sys/subsystem/net/devices/rmnet_data3<br>
sys-subsystem-net-devices-rmnet_data4.device        loaded active     plugged         /sys/subsystem/net/devices/rmnet_data4<br>
sys-subsystem-net-devices-rmnet_data5.device        loaded active     plugged         /sys/subsystem/net/devices/rmnet_data5<br>
sys-subsystem-net-devices-rmnet_data6.device        loaded active     plugged         /sys/subsystem/net/devices/rmnet_data6<br>
sys-subsystem-net-devices-rmnet_data7.device        loaded active     plugged         /sys/subsystem/net/devices/rmnet_data7<br>
sys-subsystem-net-devices-rmnet_ipa0.device         loaded active     plugged         /sys/subsystem/net/devices/rmnet_ipa0<br>
sys-subsystem-net-devices-rndis0.device             loaded active     plugged         /sys/subsystem/net/devices/rndis0<br>
sys-subsystem-net-devices-sit0.device               loaded active     plugged         /sys/subsystem/net/devices/sit0<br>
sys-subsystem-net-devices-wlan0.device              loaded active     plugged         /sys/subsystem/net/devices/wlan0<br>
sys-subsystem-rfkill-devices-rfkill2.device         loaded active     plugged         /sys/subsystem/rfkill/devices/rfkill2<br>
-.mount                                             loaded active     mounted         /<br>
config.mount                                        loaded active     mounted         Droid mount for /config<br>
data.mount                                          loaded active     mounted         /data<br>
dev-cpuset.mount                                    loaded active     mounted         Droid mount for /dev/cpuset<br>
dev-mtp.mount                                       loaded active     mounted         FFS mount<br>
dsp.mount                                           loaded active     mounted         Droid mount for /dsp<br>
firmware.mount                                      loaded active     mounted         Droid mount for /firmware<br>
mnt.mount                                           loaded active     mounted         Droid mount for /mnt<br>
persist.mount                                       loaded active     mounted         Droid mount for /persist<br>
run-state.mount                                     loaded active     mounted         /run/state<br>
run-user-100000-state.mount                         loaded active     mounted         /run/user/100000/state<br>
run-user-100000.mount                               loaded active     mounted         /run/user/100000<br>
sys-fs-fuse-connections.mount                       loaded active     mounted         FUSE Control File System<br>
sys-kernel-config.mount                             loaded active     mounted         Configuration File System<br>
sys-kernel-debug-tracing.mount                      loaded active     mounted         /sys/kernel/debug/tracing<br>
sys-kernel-debug.mount                              loaded active     mounted         Debug File System<br>
system.mount                                        loaded active     mounted         Droid mount for /system<br>
tmp.mount                                           loaded active     mounted         Temporary Directory<br>
systemd-ask-password-console.path                   loaded active     waiting         Dispatch Password Requests to Console Directory Wat<br>
systemd-ask-password-wall.path                      loaded active     waiting         Forward Password Requests to Wall Directory Watch<br>
wayland.path                                        loaded active     waiting         Wayland path watcher<br>
session-1.scope                                     loaded active     running         Session 1 of user nemo<br>
android-links.service                               loaded active     exited          Link Android folder to home<br>
autologin@100000.service                            loaded active     running         Autologin user 100000<br>
bluetooth.service                                   loaded active     running         Bluetooth service<br>
connman-vpn.service                                 loaded active     running         ConnMan VPN service<br>
connman.service                                     loaded active     running         Connection service<br>
dbus-org.freedesktop.PolicyKit1.service             loaded active     running         Authorization Manager<br>
dbus.service                                        loaded active     running         D-Bus System Message Bus<br>
droid-fm-up.service                                 loaded active     exited          Enable FM Radio<br>
droid-hal-init.service                              loaded active     running         droid-hal-init<br>
droid-hcismd-up.service                             loaded active     exited          Enable Bluetooth HCI over SMD<br>
dsme.service                                        loaded active     running         DSME<br>
init-done.service                                   loaded activating start-pre start Indicate boot is done<br>
initial-bootstate.service                           loaded active     exited          Get initial bootstate<br>
jolla-devicelock-encsfa.service                     loaded active     running         Nemo device lock daemon<br>
kmod-static-nodes.service                           loaded active     exited          Create list of required static device nodes for the<br>
ldconfig.service                                    loaded active     exited          Rebuild Dynamic Linker Cache<br>
mce.service                                         loaded active     running         Mode Control Entity (MCE)<br>
ofono.service                                       loaded active     running         Telephony service<br>
ohmd.service                                        loaded active     running         ohm daemon for resource policy management<br>
oneshot-root-late.service                           loaded inactive   dead      start Oneshot stuff for root (late run)<br>
oneshot-root.service                                loaded active     exited          Oneshot stuff for root<br>
policies-setup.service                              loaded inactive   dead      start Setup policies<br>
rescue-password-off.service                         loaded inactive   dead      start Turn rescue passwords off<br>
sensorfwd.service                                   loaded active     running         Sensor daemon for sensor framework<br>
sshd-keys.service                                   loaded active     exited          Create sshd host keys<br>
statefs.service                                     loaded active     running         StateFS FUSE filesystem, system-wide<br>
systemd-hwdb-update.service                         loaded active     exited          Rebuild Hardware Database<br>
systemd-journal-catalog-update.service              loaded active     exited          Rebuild Journal Catalog<br>
systemd-journal-flush.service                       loaded active     exited          Flush Journal to Persistent Storage<br>
systemd-journald.service                            loaded active     running         Journal Service<br>
systemd-logind.service                              loaded active     running         Login Service<br>
systemd-random-seed.service                         loaded active     exited          Load/Save Random Seed<br>
systemd-remount-fs.service                          loaded active     exited          Remount Root and Kernel File Systems<br>
systemd-sysctl.service                              loaded active     exited          Apply Kernel Variables<br>
systemd-sysusers.service                            loaded active     exited          Create System Users<br>
systemd-tmpfiles-setup-dev.service                  loaded active     exited          Create Static Device Nodes in /dev<br>
systemd-tmpfiles-setup.service                      loaded active     exited          Create Volatile Files and Directories<br>
systemd-udev-settle.service                         loaded active     exited          udev Wait for Complete Device Initialization<br>
systemd-udev-trigger.service                        loaded active     exited          udev Coldplug all Devices<br>
systemd-udevd.service                               loaded active     running         udev Kernel Device Manager<br>
systemd-update-done.service                         loaded active     exited          Update is Completed<br>
systemd-update-utmp.service                         loaded active     exited          Update UTMP about System Boot/Shutdown<br>
systemd-user-sessions.service                       loaded active     exited          Permit User Sessions<br>
systemd-vconsole-setup.service                      loaded active     exited          Setup Virtual Console<br>
udhcpd.service                                      loaded active     running         udhcpcd DHCP server<br>
udisks2.service                                     loaded active     running         Disk Manager<br>
usb-moded.service                                   loaded active     running         usb-moded USB gadget controller<br>
user@100000.service                                 loaded active     running         User Manager for UID 100000<br>
wlan-module-load.service                            loaded active     exited          Load wifi module<br>
wpa_supplicant.service                              loaded active     running         WPA Supplicant daemon<br>
-.slice                                             loaded active     active          Root Slice<br>
system-autologin.slice                              loaded active     active          system-autologin.slice<br>
system.slice                                        loaded active     active          System Slice<br>
user-100000.slice                                   loaded active     active          user-100000.slice<br>
user.slice                                          loaded active     active          User and Session Slice<br>
dbus.socket                                         loaded active     running         D-Bus System Message Bus Socket<br>
nemo-devicelock.socket                              loaded active     running         Nemo device lock socket<br>
sshd.socket                                         loaded active     listening       OpenSSH Server Socket<br>
systemd-initctl.socket                              loaded active     listening       /dev/initctl Compatibility Named Pipe<br>
systemd-journald-audit.socket                       loaded active     running         Journal Audit Socket<br>
systemd-journald-dev-log.socket                     loaded active     running         Journal Socket (/dev/log)<br>
systemd-journald.socket                             loaded active     running         Journal Socket<br>
systemd-udevd-control.socket                        loaded active     running         udev Control Socket<br>
systemd-udevd-kernel.socket                         loaded active     running         udev Kernel Socket<br>
basic.target                                        loaded active     active          Basic System<br>
bluetooth.target                                    loaded active     active          Bluetooth<br>
getty.target                                        loaded active     active          Login Prompts<br>
graphical.target                                    loaded inactive   dead      start Graphical Interface<br>
local-fs-pre.target                                 loaded active     active          Local File Systems (Pre)<br>
local-fs.target                                     loaded active     active          Local File Systems<br>
multi-user.target                                   loaded inactive   dead      start Multi-User System<br>
network.target                                      loaded active     active          Network<br>
paths.target                                        loaded active     active          Paths<br>
slices.target                                       loaded active     active          Slices<br>
sockets.target                                      loaded active     active          Sockets<br>
sound.target                                        loaded active     active          Sound Card<br>
swap.target                                         loaded active     active          Swap<br>
sysinit.target                                      loaded active     active          System Initialization<br>
timers.target                                       loaded active     active          Timers<br>
systemd-tmpfiles-clean.timer                        loaded active     waiting         Daily Cleanup of Temporary Directories<br>
<br>
LOAD   = Reflects whether the unit definition was properly loaded.<br>
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.<br>
SUB    = The low-level unit activation state, values depend on unit type.<br>
JOB    = Pending job for the unit.<br>
<br>
221 loaded units listed. Pass --all to see loaded but inactive units, too.<br>
To show all installed unit files use 'systemctl list-unit-files'.<br>
</details><br>

Посмотрим какое используется ядро в этой сборке
```console
sh-3.2# uname -a
Linux Sailfish 3.18.105-ElectraBlue-11.0-mido #3 SMP PREEMPT Fri Aug 24 19:52:43 UTC 2018 aarch64 aarch64 aarch64 GNU/Linux
```

Как видим версия ядра существенно отличается от нашей, возьмем конфиг ядра, вскоре он пригодится.
```console
sh-3.2# zcat /proc/config.gz > mido_sf_defconfig
sh-3.2# cat mido_sf_defconfig
```
<details>
#<br>
# Automatically generated file; DO NOT EDIT.<br>
# Linux/arm64 3.18.105 Kernel Configuration<br>
#<br>
CONFIG_ARM64=y<br>
CONFIG_64BIT=y<br>
CONFIG_ARCH_PHYS_ADDR_T_64BIT=y<br>
CONFIG_MMU=y<br>
CONFIG_ARCH_MMAP_RND_BITS_MIN=18<br>
CONFIG_ARCH_MMAP_RND_BITS_MAX=24<br>
CONFIG_ARCH_MMAP_RND_COMPAT_BITS_MIN=11<br>
CONFIG_ARCH_MMAP_RND_COMPAT_BITS_MAX=16<br>
CONFIG_ILLEGAL_POINTER_VALUE=0xdead000000000000<br>
CONFIG_STACKTRACE_SUPPORT=y<br>
CONFIG_LOCKDEP_SUPPORT=y<br>
CONFIG_TRACE_IRQFLAGS_SUPPORT=y<br>
CONFIG_RWSEM_XCHGADD_ALGORITHM=y<br>
CONFIG_GENERIC_BUG=y<br>
CONFIG_GENERIC_HWEIGHT=y<br>
CONFIG_GENERIC_CSUM=y<br>
CONFIG_GENERIC_CALIBRATE_DELAY=y<br>
CONFIG_ZONE_DMA=y<br>
CONFIG_HAVE_GENERIC_RCU_GUP=y<br>
CONFIG_ARCH_DMA_ADDR_T_64BIT=y<br>
CONFIG_NEED_DMA_MAP_STATE=y<br>
CONFIG_NEED_SG_DMA_LENGTH=y<br>
CONFIG_ARM64_DMA_USE_IOMMU=y<br>
CONFIG_ARM64_DMA_IOMMU_ALIGNMENT=8<br>
CONFIG_SWIOTLB=y<br>
CONFIG_IOMMU_HELPER=y<br>
CONFIG_KERNEL_MODE_NEON=y<br>
CONFIG_FIX_EARLYCON_MEM=y<br>
CONFIG_PGTABLE_LEVELS=3<br>
CONFIG_DEFCONFIG_LIST="/lib/modules/$UNAME_RELEASE/.config"<br>
CONFIG_IRQ_WORK=y<br>
CONFIG_BUILDTIME_EXTABLE_SORT=y<br>
<br>
#<br>
# General setup<br>
#<br>
CONFIG_INIT_ENV_ARG_LIMIT=32<br>
CONFIG_CROSS_COMPILE=""<br>
# CONFIG_COMPILE_TEST is not set<br>
CONFIG_LOCALVERSION="-ElectraBlue-11.0-mido"<br>
# CONFIG_LOCALVERSION_AUTO is not set<br>
CONFIG_DEFAULT_HOSTNAME="(none)"<br>
CONFIG_SWAP=y<br>
CONFIG_SYSVIPC=y<br>
CONFIG_SYSVIPC_SYSCTL=y<br>
# CONFIG_POSIX_MQUEUE is not set<br>
CONFIG_CROSS_MEMORY_ATTACH=y<br>
CONFIG_FHANDLE=y<br>
CONFIG_USELIB=y<br>
CONFIG_AUDIT=y<br>
CONFIG_HAVE_ARCH_AUDITSYSCALL=y<br>
CONFIG_AUDITSYSCALL=y<br>
CONFIG_AUDIT_WATCH=y<br>
CONFIG_AUDIT_TREE=y<br>
<br>
#<br>
# IRQ subsystem<br>
#<br>
CONFIG_GENERIC_IRQ_PROBE=y<br>
CONFIG_GENERIC_IRQ_SHOW=y<br>
CONFIG_HARDIRQS_SW_RESEND=y<br>
CONFIG_IRQ_DOMAIN=y<br>
CONFIG_IRQ_DOMAIN_HIERARCHY=y<br>
CONFIG_GENERIC_MSI_IRQ=y<br>
CONFIG_GENERIC_MSI_IRQ_DOMAIN=y<br>
CONFIG_HANDLE_DOMAIN_IRQ=y<br>
# CONFIG_IRQ_DOMAIN_DEBUG is not set<br>
CONFIG_SPARSE_IRQ=y<br>
CONFIG_GENERIC_TIME_VSYSCALL=y<br>
CONFIG_GENERIC_CLOCKEVENTS=y<br>
CONFIG_GENERIC_CLOCKEVENTS_BUILD=y<br>
CONFIG_ARCH_HAS_TICK_BROADCAST=y<br>
CONFIG_GENERIC_CLOCKEVENTS_BROADCAST=y<br>
<br>
#<br>
# Timers subsystem<br>
#<br>
CONFIG_TICK_ONESHOT=y<br>
CONFIG_NO_HZ_COMMON=y<br>
# CONFIG_HZ_PERIODIC is not set<br>
CONFIG_NO_HZ_IDLE=y<br>
# CONFIG_NO_HZ_FULL is not set<br>
CONFIG_NO_HZ=y<br>
CONFIG_HIGH_RES_TIMERS=y<br>
<br>
#<br>
# CPU/Task time and stats accounting<br>
#<br>
# CONFIG_TICK_CPU_ACCOUNTING is not set<br>
# CONFIG_VIRT_CPU_ACCOUNTING_GEN is not set<br>
CONFIG_IRQ_TIME_ACCOUNTING=y<br>
# CONFIG_BSD_PROCESS_ACCT is not set<br>
# CONFIG_TASKSTATS is not set<br>
<br>
#<br>
# RCU Subsystem<br>
#<br>
CONFIG_TREE_PREEMPT_RCU=y<br>
CONFIG_PREEMPT_RCU=y<br>
# CONFIG_TASKS_RCU is not set<br>
CONFIG_RCU_STALL_COMMON=y<br>
# CONFIG_RCU_USER_QS is not set<br>
CONFIG_RCU_FANOUT=64<br>
CONFIG_RCU_FANOUT_LEAF=16<br>
# CONFIG_RCU_FANOUT_EXACT is not set<br>
CONFIG_RCU_FAST_NO_HZ=y<br>
# CONFIG_TREE_RCU_TRACE is not set<br>
# CONFIG_RCU_BOOST is not set<br>
CONFIG_RCU_NOCB_CPU=y<br>
# CONFIG_RCU_NOCB_CPU_NONE is not set<br>
# CONFIG_RCU_NOCB_CPU_ZERO is not set<br>
CONFIG_RCU_NOCB_CPU_ALL=y<br>
CONFIG_BUILD_BIN2C=y<br>
CONFIG_IKCONFIG=y<br>
CONFIG_IKCONFIG_PROC=y<br>
CONFIG_LOG_BUF_SHIFT=17<br>
# CONFIG_CONSOLE_FLUSH_ON_HOTPLUG is not set<br>
CONFIG_LOG_CPU_MAX_BUF_SHIFT=12<br>
CONFIG_GENERIC_SCHED_CLOCK=y<br>
CONFIG_CGROUPS=y<br>
# CONFIG_CGROUP_DEBUG is not set<br>
CONFIG_CGROUP_FREEZER=y<br>
CONFIG_CGROUP_DEVICE=y<br>
CONFIG_CPUSETS=y<br>
CONFIG_PROC_PID_CPUSET=y<br>
CONFIG_CGROUP_CPUACCT=y<br>
CONFIG_RESOURCE_COUNTERS=y<br>
CONFIG_MEMCG=y<br>
CONFIG_MEMCG_SWAP=y<br>
CONFIG_MEMCG_SWAP_ENABLED=y<br>
CONFIG_MEMCG_KMEM=y<br>
CONFIG_CGROUP_PERF=y<br>
CONFIG_CGROUP_SCHED=y<br>
CONFIG_FAIR_GROUP_SCHED=y<br>
CONFIG_CFS_BANDWIDTH=y<br>
CONFIG_RT_GROUP_SCHED=y<br>
CONFIG_BLK_CGROUP=y<br>
# CONFIG_DEBUG_BLK_CGROUP is not set<br>
CONFIG_SCHED_HMP=y<br>
# CONFIG_SCHED_HMP_CSTATE_AWARE is not set<br>
# CONFIG_SCHED_CORE_CTL is not set<br>
# CONFIG_SCHED_QHMP is not set<br>
CONFIG_CHECKPOINT_RESTORE=y<br>
CONFIG_NAMESPACES=y<br>
CONFIG_UTS_NS=y<br>
CONFIG_IPC_NS=y<br>
CONFIG_USER_NS=y<br>
CONFIG_PID_NS=y<br>
CONFIG_NET_NS=y<br>
# CONFIG_SCHED_AUTOGROUP is not set<br>
# CONFIG_SYSFS_DEPRECATED is not set<br>
# CONFIG_RELAY is not set<br>
CONFIG_BLK_DEV_INITRD=y<br>
CONFIG_INITRAMFS_SOURCE=""<br>
CONFIG_RD_GZIP=y<br>
CONFIG_RD_BZIP2=y<br>
CONFIG_RD_LZMA=y<br>
# CONFIG_RD_XZ is not set<br>
# CONFIG_RD_LZO is not set<br>
# CONFIG_RD_LZ4 is not set<br>
CONFIG_CC_OPTIMIZE_FOR_SIZE=y<br>
CONFIG_SYSCTL=y<br>
CONFIG_ANON_INODES=y<br>
CONFIG_HAVE_UID16=y<br>
CONFIG_SYSCTL_EXCEPTION_TRACE=y<br>
CONFIG_BPF=y<br>
CONFIG_EXPERT=y<br>
CONFIG_UID16=y<br>
# CONFIG_SGETMASK_SYSCALL is not set<br>
CONFIG_SYSFS_SYSCALL=y<br>
# CONFIG_SYSCTL_SYSCALL is not set<br>
CONFIG_KALLSYMS=y<br>
CONFIG_KALLSYMS_ALL=y<br>
CONFIG_PRINTK=y<br>
CONFIG_BUG=y<br>
CONFIG_ELF_CORE=y<br>
CONFIG_BASE_FULL=y<br>
CONFIG_FUTEX=y<br>
CONFIG_EPOLL=y<br>
CONFIG_SIGNALFD=y<br>
CONFIG_TIMERFD=y<br>
CONFIG_EVENTFD=y<br>
# CONFIG_BPF_SYSCALL is not set<br>
CONFIG_SHMEM=y<br>
CONFIG_AIO=y<br>
CONFIG_ADVISE_SYSCALLS=y<br>
CONFIG_PCI_QUIRKS=y<br>
CONFIG_EMBEDDED=y<br>
CONFIG_HAVE_PERF_EVENTS=y<br>
CONFIG_PERF_USE_VMALLOC=y<br>
<br>
#<br>
# Kernel Performance Events And Counters<br>
#<br>
CONFIG_PERF_EVENTS=y<br>
# CONFIG_DEBUG_PERF_USE_VMALLOC is not set<br>
CONFIG_VM_EVENT_COUNTERS=y<br>
# CONFIG_SLUB_DEBUG is not set<br>
CONFIG_COMPAT_BRK=y<br>
# CONFIG_SLAB is not set<br>
CONFIG_SLUB=y<br>
# CONFIG_SLOB is not set<br>
CONFIG_SLUB_CPU_PARTIAL=y<br>
CONFIG_SYSTEM_TRUSTED_KEYRING=y<br>
CONFIG_PROFILING=y<br>
CONFIG_TRACEPOINTS=y<br>
# CONFIG_JUMP_LABEL is not set<br>
# CONFIG_UPROBES is not set<br>
# CONFIG_HAVE_64BIT_ALIGNED_ACCESS is not set<br>
CONFIG_HAVE_EFFICIENT_UNALIGNED_ACCESS=y<br>
CONFIG_HAVE_ARCH_TRACEHOOK=y<br>
CONFIG_HAVE_DMA_ATTRS=y<br>
CONFIG_HAVE_DMA_CONTIGUOUS=y<br>
CONFIG_GENERIC_SMP_IDLE_THREAD=y<br>
CONFIG_HAVE_CLK=y<br>
CONFIG_HAVE_DMA_API_DEBUG=y<br>
CONFIG_HAVE_PERF_REGS=y<br>
CONFIG_HAVE_PERF_USER_STACK_DUMP=y<br>
CONFIG_HAVE_ARCH_JUMP_LABEL=y<br>
CONFIG_HAVE_RCU_TABLE_FREE=y<br>
CONFIG_HAVE_ALIGNED_STRUCT_PAGE=y<br>
CONFIG_HAVE_CMPXCHG_DOUBLE=y<br>
CONFIG_ARCH_WANT_COMPAT_IPC_PARSE_VERSION=y<br>
CONFIG_HAVE_ARCH_SECCOMP_FILTER=y<br>
CONFIG_SECCOMP_FILTER=y<br>
CONFIG_HAVE_CC_STACKPROTECTOR=y<br>
CONFIG_CC_STACKPROTECTOR=y<br>
# CONFIG_CC_STACKPROTECTOR_NONE is not set<br>
# CONFIG_CC_STACKPROTECTOR_REGULAR is not set<br>
CONFIG_CC_STACKPROTECTOR_STRONG=y<br>
CONFIG_HAVE_CONTEXT_TRACKING=y<br>
CONFIG_HAVE_VIRT_CPU_ACCOUNTING_GEN=y<br>
CONFIG_HAVE_IRQ_TIME_ACCOUNTING=y<br>
CONFIG_HAVE_ARCH_TRANSPARENT_HUGEPAGE=y<br>
CONFIG_MODULES_USE_ELF_RELA=y<br>
CONFIG_HAVE_ARCH_MMAP_RND_BITS=y<br>
CONFIG_ARCH_MMAP_RND_BITS=18<br>
CONFIG_HAVE_ARCH_MMAP_RND_COMPAT_BITS=y<br>
CONFIG_ARCH_MMAP_RND_COMPAT_BITS=16<br>
CONFIG_CLONE_BACKWARDS=y<br>
CONFIG_OLD_SIGSUSPEND3=y<br>
CONFIG_COMPAT_OLD_SIGACTION=y<br>
<br>
#<br>
# GCOV-based kernel profiling<br>
#<br>
# CONFIG_GCOV_KERNEL is not set<br>
CONFIG_HAVE_GENERIC_DMA_COHERENT=y<br>
CONFIG_RT_MUTEXES=y<br>
CONFIG_BASE_SMALL=0<br>
CONFIG_MODULES=y<br>
# CONFIG_MODULE_FORCE_LOAD is not set<br>
CONFIG_MODULE_UNLOAD=y<br>
CONFIG_MODULE_FORCE_UNLOAD=y<br>
CONFIG_MODVERSIONS=y<br>
# CONFIG_MODULE_SRCVERSION_ALL is not set<br>
# CONFIG_MODULE_SIG is not set<br>
# CONFIG_MODULE_COMPRESS is not set<br>
CONFIG_STOP_MACHINE=y<br>
CONFIG_BLOCK=y<br>
CONFIG_BLK_DEV_BSG=y<br>
# CONFIG_BLK_DEV_BSGLIB is not set<br>
# CONFIG_BLK_DEV_INTEGRITY is not set<br>
# CONFIG_BLK_DEV_THROTTLING is not set<br>
# CONFIG_BLK_CMDLINE_PARSER is not set<br>
<br>
#<br>
# Partition Types<br>
#<br>
CONFIG_PARTITION_ADVANCED=y<br>
# CONFIG_ACORN_PARTITION is not set<br>
# CONFIG_AIX_PARTITION is not set<br>
# CONFIG_OSF_PARTITION is not set<br>
# CONFIG_AMIGA_PARTITION is not set<br>
# CONFIG_ATARI_PARTITION is not set<br>
# CONFIG_MAC_PARTITION is not set<br>
CONFIG_MSDOS_PARTITION=y<br>
# CONFIG_BSD_DISKLABEL is not set<br>
# CONFIG_MINIX_SUBPARTITION is not set<br>
# CONFIG_SOLARIS_X86_PARTITION is not set<br>
# CONFIG_UNIXWARE_DISKLABEL is not set<br>
# CONFIG_LDM_PARTITION is not set<br>
# CONFIG_SGI_PARTITION is not set<br>
# CONFIG_ULTRIX_PARTITION is not set<br>
# CONFIG_SUN_PARTITION is not set<br>
# CONFIG_KARMA_PARTITION is not set<br>
CONFIG_EFI_PARTITION=y<br>
# CONFIG_SYSV68_PARTITION is not set<br>
# CONFIG_CMDLINE_PARTITION is not set<br>
CONFIG_BLOCK_COMPAT=y<br>
<br>
#<br>
# IO Schedulers<br>
#<br>
CONFIG_IOSCHED_NOOP=y<br>
# CONFIG_IOSCHED_TEST is not set<br>
CONFIG_IOSCHED_DEADLINE=y<br>
CONFIG_IOSCHED_MAPLE=y<br>
CONFIG_IOSCHED_CFQ=y<br>
CONFIG_IOSCHED_ZEN=y<br>
CONFIG_IOSCHED_FIOPS=y<br>
CONFIG_IOSCHED_SIO=y<br>
# CONFIG_CFQ_GROUP_IOSCHED is not set<br>
CONFIG_IOSCHED_SWITCHER=y<br>
CONFIG_IOSCHED_BFQ=y<br>
# CONFIG_CGROUP_BFQIO is not set<br>
# CONFIG_DEFAULT_DEADLINE is not set<br>
# CONFIG_DEFAULT_CFQ is not set<br>
CONFIG_DEFAULT_MAPLE=y<br>
# CONFIG_DEFAULT_BFQ is not set<br>
# CONFIG_DEFAULT_NOOP is not set<br>
# CONFIG_DEFAULT_ZEN is not set<br>
# CONFIG_DEFAULT_SIO is not set<br>
# CONFIG_DEFAULT_FIOPS is not set<br>
CONFIG_DEFAULT_IOSCHED="maple"<br>
CONFIG_ASN1=y<br>
CONFIG_UNINLINE_SPIN_UNLOCK=y<br>
CONFIG_ARCH_SUPPORTS_ATOMIC_RMW=y<br>
CONFIG_MUTEX_SPIN_ON_OWNER=y<br>
CONFIG_RWSEM_SPIN_ON_OWNER=y<br>
CONFIG_FREEZER=y<br>
<br>
#<br>
# Platform selection<br>
#<br>
# CONFIG_ARCH_THUNDER is not set<br>
# CONFIG_ARCH_VEXPRESS is not set<br>
# CONFIG_ARCH_XGENE is not set<br>
CONFIG_ARCH_MSM=y<br>
CONFIG_ARCH_MSM8916=y<br>
# CONFIG_ARCH_MSM8917 is not set<br>
# CONFIG_ARCH_MSM8920 is not set<br>
# CONFIG_ARCH_MSM8940 is not set<br>
CONFIG_ARCH_MSM8953=y<br>
# CONFIG_ARCH_SDM450 is not set<br>
# CONFIG_ARCH_MSM8937 is not set<br>
# CONFIG_ARCH_MSM8996 is not set<br>
# CONFIG_ARCH_MSMCOBALT is not set<br>
<br>
#<br>
# Bus support<br>
#<br>
CONFIG_ARM_AMBA=y<br>
CONFIG_PCI=y<br>
CONFIG_PCI_DOMAINS=y<br>
CONFIG_PCI_DOMAINS_GENERIC=y<br>
CONFIG_PCI_SYSCALL=y<br>
CONFIG_PCI_MSI=y<br>
CONFIG_PCI_MSI_IRQ_DOMAIN=y<br>
# CONFIG_PCI_DEBUG is not set<br>
# CONFIG_PCI_REALLOC_ENABLE_AUTO is not set<br>
# CONFIG_PCI_STUB is not set<br>
# CONFIG_PCI_IOV is not set<br>
# CONFIG_PCI_PRI is not set<br>
# CONFIG_PCI_PASID is not set<br>
CONFIG_PCI_MSM=y<br>
CONFIG_PCI_LABEL=y<br>
<br>
#<br>
# PCI host controller drivers<br>
#<br>
# CONFIG_PCI_HOST_GENERIC is not set<br>
# CONFIG_PCIEPORTBUS is not set<br>
# CONFIG_HOTPLUG_PCI is not set<br>
<br>
#<br>
# Kernel Features<br>
#<br>
<br>
#<br>
# ARM errata workarounds via the alternatives framework<br>
#<br>
CONFIG_ARM64_ERRATUM_826319=y<br>
CONFIG_ARM64_ERRATUM_827319=y<br>
CONFIG_ARM64_ERRATUM_824069=y<br>
CONFIG_ARM64_ERRATUM_819472=y<br>
CONFIG_ARM64_ERRATUM_832075=y<br>
CONFIG_ARM64_ERRATUM_845719=y<br>
CONFIG_ARM64_4K_PAGES=y<br>
# CONFIG_ARM64_64K_PAGES is not set<br>
# CONFIG_ARM64_DCACHE_DISABLE is not set<br>
# CONFIG_ARM64_ICACHE_DISABLE is not set<br>
CONFIG_ARCH_MSM8953_SOC_SETTINGS=y<br>
CONFIG_ARM64_VA_BITS_39=y<br>
CONFIG_ARM64_VA_BITS=39<br>
CONFIG_ARM64_PGTABLE_LEVELS=3<br>
# CONFIG_CPU_BIG_ENDIAN is not set<br>
# CONFIG_ARM64_SEV_IN_LOCK_UNLOCK is not set<br>
CONFIG_SMP=y<br>
CONFIG_SCHED_MC=y<br>
# CONFIG_SCHED_SMT is not set<br>
CONFIG_NR_CPUS=8<br>
CONFIG_HOTPLUG_CPU=y<br>
CONFIG_ARCH_NR_GPIO=1024<br>
# CONFIG_PREEMPT_NONE is not set<br>
# CONFIG_PREEMPT_VOLUNTARY is not set<br>
CONFIG_PREEMPT=y<br>
CONFIG_PREEMPT_COUNT=y<br>
# CONFIG_HZ_100 is not set<br>
CONFIG_HZ_250=y<br>
# CONFIG_HZ_300 is not set<br>
# CONFIG_HZ_1000 is not set<br>
CONFIG_HZ=250<br>
CONFIG_SCHED_HRTICK=y<br>
CONFIG_ARCH_HAS_HOLES_MEMORYMODEL=y<br>
CONFIG_ARCH_SPARSEMEM_ENABLE=y<br>
CONFIG_ARCH_SPARSEMEM_DEFAULT=y<br>
CONFIG_ARCH_SELECT_MEMORY_MODEL=y<br>
CONFIG_HAVE_ARCH_PFN_VALID=y<br>
CONFIG_HW_PERF_EVENTS=y<br>
# CONFIG_PERF_EVENTS_USERMODE is not set<br>
# CONFIG_PERF_EVENTS_RESET_PMU_DEBUGFS is not set<br>
# CONFIG_ARM64_REG_REBALANCE_ON_CTX_SW is not set<br>
CONFIG_SYS_SUPPORTS_HUGETLBFS=y<br>
CONFIG_ARCH_WANT_GENERAL_HUGETLB=y<br>
CONFIG_ARCH_WANT_HUGE_PMD_SHARE=y<br>
CONFIG_ARCH_HAS_CACHE_LINE_SIZE=y<br>
CONFIG_SELECT_MEMORY_MODEL=y<br>
CONFIG_SPARSEMEM_MANUAL=y<br>
CONFIG_SPARSEMEM=y<br>
CONFIG_HAVE_MEMORY_PRESENT=y<br>
CONFIG_SPARSEMEM_EXTREME=y<br>
CONFIG_SPARSEMEM_VMEMMAP_ENABLE=y<br>
CONFIG_SPARSEMEM_VMEMMAP=y<br>
CONFIG_HAVE_MEMBLOCK=y<br>
CONFIG_NO_BOOTMEM=y<br>
CONFIG_MEMORY_ISOLATION=y<br>
# CONFIG_HAVE_BOOTMEM_INFO_NODE is not set<br>
CONFIG_PAGEFLAGS_EXTENDED=y<br>
CONFIG_SPLIT_PTLOCK_CPUS=4<br>
CONFIG_COMPACTION=y<br>
CONFIG_MIGRATION=y<br>
CONFIG_PHYS_ADDR_T_64BIT=y<br>
CONFIG_ZONE_DMA_FLAG=1<br>
CONFIG_BOUNCE=y<br>
CONFIG_KSM=y<br>
CONFIG_DEFAULT_MMAP_MIN_ADDR=4096<br>
# CONFIG_TRANSPARENT_HUGEPAGE is not set<br>
CONFIG_CLEANCACHE=y<br>
# CONFIG_FRONTSWAP is not set<br>
CONFIG_CMA=y<br>
# CONFIG_CMA_DEBUG is not set<br>
CONFIG_CMA_DEBUGFS=y<br>
CONFIG_CMA_AREAS=7<br>
# CONFIG_ZPOOL is not set<br>
CONFIG_ZBUD=y<br>
CONFIG_ZSMALLOC=y<br>
# CONFIG_PGTABLE_MAPPING is not set<br>
# CONFIG_ZSMALLOC_STAT is not set<br>
CONFIG_GENERIC_EARLY_IOREMAP=y<br>
CONFIG_ZCACHE=y<br>
CONFIG_BALANCE_ANON_FILE_RECLAIM=y<br>
CONFIG_KSWAPD_CPU_AFFINITY_MASK=""<br>
# CONFIG_FORCE_ALLOC_FROM_DMA_ZONE is not set<br>
CONFIG_PROCESS_RECLAIM=y<br>
CONFIG_SECCOMP=y<br>
# CONFIG_XEN is not set<br>
CONFIG_FORCE_MAX_ZONEORDER=11<br>
CONFIG_ARM64_PAN=y<br>
CONFIG_ARMV8_DEPRECATED=y<br>
CONFIG_SWP_EMULATION=y<br>
CONFIG_CP15_BARRIER_EMULATION=y<br>
CONFIG_SETEND_EMULATION=y<br>
<br>
#<br>
# ARMv8.1 architectural features<br>
#<br>
CONFIG_ARM64_UAO=y<br>
<br>
#<br>
# Boot options<br>
#<br>
CONFIG_CMDLINE=""<br>
CONFIG_EFI_STUB=y<br>
CONFIG_EFI=y<br>
CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE=y<br>
CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE_NAMES=""<br>
CONFIG_DMI=y<br>
<br>
#<br>
# Userspace binary formats<br>
#<br>
CONFIG_BINFMT_ELF=y<br>
CONFIG_COMPAT_BINFMT_ELF=y<br>
# CONFIG_CORE_DUMP_DEFAULT_ELF_HEADERS is not set<br>
CONFIG_BINFMT_SCRIPT=y<br>
# CONFIG_HAVE_AOUT is not set<br>
# CONFIG_BINFMT_MISC is not set<br>
CONFIG_COREDUMP=y<br>
CONFIG_COMPAT=y<br>
CONFIG_SYSVIPC_COMPAT=y<br>
<br>
#<br>
# Power management options<br>
#<br>
CONFIG_SUSPEND=y<br>
CONFIG_SUSPEND_FREEZER=y<br>
CONFIG_WAKELOCK=y<br>
CONFIG_POWERSUSPEND=y<br>
# CONFIG_POWERSUSPEND_DEBUG is not set<br>
CONFIG_PM_SLEEP=y<br>
CONFIG_PM_SLEEP_SMP=y<br>
CONFIG_PM_AUTOSLEEP=y<br>
CONFIG_PM_WAKELOCKS=y<br>
CONFIG_PM_WAKELOCKS_LIMIT=0<br>
# CONFIG_PM_WAKELOCKS_GC is not set<br>
CONFIG_PM_RUNTIME=y<br>
CONFIG_PM=y<br>
CONFIG_PM_DEBUG=y<br>
# CONFIG_PM_ADVANCED_DEBUG is not set<br>
# CONFIG_PM_TEST_SUSPEND is not set<br>
CONFIG_PM_SLEEP_DEBUG=y<br>
# CONFIG_DPM_WATCHDOG is not set<br>
CONFIG_PM_OPP=y<br>
CONFIG_PM_CLK=y<br>
# CONFIG_WQ_POWER_EFFICIENT_DEFAULT is not set<br>
CONFIG_CPU_PM=y<br>
CONFIG_SUSPEND_TIME=y<br>
# CONFIG_QUICK_WAKEUP is not set<br>
CONFIG_BOEFFLA_WL_BLOCKER=y<br>
CONFIG_ARCH_SUSPEND_POSSIBLE=y<br>
CONFIG_ARM64_CPU_SUSPEND=y<br>
<br>
#<br>
# CPU Power Management<br>
#<br>
<br>
#<br>
# CPU Idle<br>
#<br>
CONFIG_CPU_IDLE=y<br>
CONFIG_CPU_IDLE_MULTIPLE_DRIVERS=y<br>
CONFIG_CPU_IDLE_GOV_LADDER=y<br>
CONFIG_CPU_IDLE_GOV_MENU=y<br>
<br>
#<br>
# ARM64 CPU Idle Drivers<br>
#<br>
# CONFIG_ARM64_CPUIDLE is not set<br>
# CONFIG_ARCH_NEEDS_CPU_IDLE_COUPLED is not set<br>
<br>
#<br>
# CPU Frequency scaling<br>
#<br>
CONFIG_CPU_FREQ=y<br>
CONFIG_CPU_FREQ_GOV_COMMON=y<br>
CONFIG_SCHED_FREQ_INPUT=y<br>
CONFIG_CPU_FREQ_STAT=y<br>
# CONFIG_CPU_FREQ_STAT_DETAILS is not set<br>
CONFIG_CPU_FREQ_DEFAULT_GOV_PERFORMANCE=y<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_POWERSAVE is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_USERSPACE is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_ONDEMAND is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_CONSERVATIVE is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_ZZMOOVE is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_INTERACTIVE is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_BLU_ACTIVE is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_NIGHTMARE is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_DARKNESS is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_ALUCARD is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_RELAXED is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_CHILL is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_ELECTRON is not set<br>
CONFIG_CPU_FREQ_GOV_PERFORMANCE=y<br>
CONFIG_CPU_FREQ_GOV_POWERSAVE=y<br>
CONFIG_CPU_FREQ_GOV_USERSPACE=y<br>
CONFIG_CPU_FREQ_GOV_ONDEMAND=y<br>
CONFIG_CPU_FREQ_GOV_ZZMOOVE=y<br>
CONFIG_CPU_FREQ_GOV_INTERACTIVE=y<br>
CONFIG_CPU_FREQ_GOV_CHILL=y<br>
CONFIG_CPU_FREQ_GOV_NIGHTMARE=y<br>
CONFIG_CPU_FREQ_GOV_DARKNESS=y<br>
CONFIG_CPU_FREQ_GOV_ALUCARD=y<br>
# CONFIG_CPUFREQ_DT is not set<br>
CONFIG_CPU_FREQ_GOV_RELAXED=y<br>
CONFIG_CPU_FREQ_GOV_ELECTRON=y<br>
# CONFIG_CPU_BOOST is not set<br>
CONFIG_CPU_FREQ_GOV_BLU_ACTIVE=y<br>
CONFIG_MSM_TRACK_FREQ_TARGET_INDEX=y<br>
CONFIG_FINGERPRINT_BOOST=y<br>
<br>
#<br>
# ARM CPU frequency scaling drivers<br>
#<br>
# CONFIG_ARM_KIRKWOOD_CPUFREQ is not set<br>
CONFIG_CPU_FREQ_MSM=y<br>
CONFIG_ARM64_ERRATUM_843419=y<br>
CONFIG_NET=y<br>
CONFIG_COMPAT_NETLINK_MESSAGES=y<br>
# CONFIG_DISABLE_NET_SKB_FRAG_CACHE is not set<br>
<br>
#<br>
# Networking options<br>
#<br>
CONFIG_PACKET=y<br>
CONFIG_PACKET_DIAG=y<br>
CONFIG_UNIX=y<br>
CONFIG_UNIX_DIAG=y<br>
CONFIG_XFRM=y<br>
CONFIG_XFRM_ALGO=y<br>
CONFIG_XFRM_USER=y<br>
# CONFIG_XFRM_SUB_POLICY is not set<br>
# CONFIG_XFRM_MIGRATE is not set<br>
CONFIG_XFRM_STATISTICS=y<br>
CONFIG_XFRM_IPCOMP=y<br>
CONFIG_NET_KEY=y<br>
# CONFIG_NET_KEY_MIGRATE is not set<br>
CONFIG_INET=y<br>
CONFIG_WIREGUARD=y<br>
# CONFIG_WIREGUARD_DEBUG is not set<br>
CONFIG_IP_MULTICAST=y<br>
CONFIG_IP_ADVANCED_ROUTER=y<br>
# CONFIG_IP_FIB_TRIE_STATS is not set<br>
CONFIG_IP_MULTIPLE_TABLES=y<br>
# CONFIG_IP_ROUTE_MULTIPATH is not set<br>
CONFIG_IP_ROUTE_VERBOSE=y<br>
CONFIG_IP_PNP=y<br>
CONFIG_IP_PNP_DHCP=y<br>
# CONFIG_IP_PNP_BOOTP is not set<br>
# CONFIG_IP_PNP_RARP is not set<br>
# CONFIG_NET_IPIP is not set<br>
# CONFIG_NET_IPGRE_DEMUX is not set<br>
CONFIG_NET_IP_TUNNEL=y<br>
# CONFIG_IP_MROUTE is not set<br>
# CONFIG_SYN_COOKIES is not set<br>
# CONFIG_NET_IPVTI is not set<br>
CONFIG_NET_UDP_TUNNEL=y<br>
# CONFIG_NET_FOU is not set<br>
# CONFIG_GENEVE is not set<br>
CONFIG_INET_AH=y<br>
CONFIG_INET_ESP=y<br>
CONFIG_INET_IPCOMP=y<br>
CONFIG_INET_XFRM_TUNNEL=y<br>
CONFIG_INET_TUNNEL=y<br>
CONFIG_INET_XFRM_MODE_TRANSPORT=y<br>
CONFIG_INET_XFRM_MODE_TUNNEL=y<br>
# CONFIG_INET_XFRM_MODE_BEET is not set<br>
# CONFIG_INET_LRO is not set<br>
CONFIG_INET_DIAG=y<br>
CONFIG_INET_TCP_DIAG=y<br>
# CONFIG_INET_UDP_DIAG is not set<br>
CONFIG_INET_DIAG_DESTROY=y<br>
CONFIG_TCP_CONG_ADVANCED=y<br>
CONFIG_TCP_CONG_BIC=m<br>
CONFIG_TCP_CONG_CUBIC=y<br>
CONFIG_TCP_CONG_WESTWOOD=m<br>
CONFIG_TCP_CONG_HTCP=y<br>
CONFIG_TCP_CONG_HSTCP=y<br>
# CONFIG_TCP_CONG_HYBLA is not set<br>
CONFIG_TCP_CONG_VEGAS=y<br>
CONFIG_TCP_CONG_SCALABLE=y<br>
CONFIG_TCP_CONG_LP=y<br>
CONFIG_TCP_CONG_VENO=y<br>
CONFIG_TCP_CONG_YEAH=y<br>
CONFIG_TCP_CONG_ILLINOIS=y<br>
# CONFIG_TCP_CONG_DCTCP is not set<br>
CONFIG_DEFAULT_CUBIC=y<br>
# CONFIG_DEFAULT_HTCP is not set<br>
# CONFIG_DEFAULT_VEGAS is not set<br>
# CONFIG_DEFAULT_VENO is not set<br>
# CONFIG_DEFAULT_RENO is not set<br>
CONFIG_DEFAULT_TCP_CONG="cubic"<br>
# CONFIG_TCP_MD5SIG is not set<br>
CONFIG_IPV6=y<br>
CONFIG_IPV6_ROUTER_PREF=y<br>
CONFIG_IPV6_ROUTE_INFO=y<br>
CONFIG_IPV6_OPTIMISTIC_DAD=y<br>
CONFIG_INET6_AH=y<br>
CONFIG_INET6_ESP=y<br>
CONFIG_INET6_IPCOMP=y<br>
CONFIG_IPV6_MIP6=y<br>
CONFIG_INET6_XFRM_TUNNEL=y<br>
CONFIG_INET6_TUNNEL=y<br>
CONFIG_INET6_XFRM_MODE_TRANSPORT=y<br>
CONFIG_INET6_XFRM_MODE_TUNNEL=y<br>
CONFIG_INET6_XFRM_MODE_BEET=y<br>
# CONFIG_INET6_XFRM_MODE_ROUTEOPTIMIZATION is not set<br>
# CONFIG_IPV6_VTI is not set<br>
CONFIG_IPV6_SIT=y<br>
# CONFIG_IPV6_SIT_6RD is not set<br>
CONFIG_IPV6_NDISC_NODETYPE=y<br>
# CONFIG_IPV6_TUNNEL is not set<br>
# CONFIG_IPV6_GRE is not set<br>
CONFIG_IPV6_MULTIPLE_TABLES=y<br>
CONFIG_IPV6_SUBTREES=y<br>
# CONFIG_IPV6_MROUTE is not set<br>
# CONFIG_NETLABEL is not set<br>
CONFIG_ANDROID_PARANOID_NETWORK=y<br>
CONFIG_NET_ACTIVITY_STATS=y<br>
CONFIG_NETWORK_SECMARK=y<br>
# CONFIG_NET_PTP_CLASSIFY is not set<br>
# CONFIG_NETWORK_PHY_TIMESTAMPING is not set<br>
CONFIG_NETFILTER=y<br>
# CONFIG_NETFILTER_DEBUG is not set<br>
CONFIG_NETFILTER_ADVANCED=y<br>
CONFIG_BRIDGE_NETFILTER=y<br>
<br>
#<br>
# Core Netfilter Configuration<br>
#<br>
CONFIG_NETFILTER_NETLINK=y<br>
CONFIG_NETFILTER_NETLINK_ACCT=m<br>
CONFIG_NETFILTER_NETLINK_QUEUE=y<br>
CONFIG_NETFILTER_NETLINK_LOG=y<br>
CONFIG_NF_CONNTRACK=y<br>
CONFIG_NF_LOG_COMMON=y<br>
CONFIG_NF_CONNTRACK_MARK=y<br>
CONFIG_NF_CONNTRACK_SECMARK=y<br>
# CONFIG_NF_CONNTRACK_ZONES is not set<br>
CONFIG_NF_CONNTRACK_PROCFS=y<br>
CONFIG_NF_CONNTRACK_EVENTS=y<br>
# CONFIG_NF_CONNTRACK_TIMEOUT is not set<br>
# CONFIG_NF_CONNTRACK_TIMESTAMP is not set<br>
CONFIG_NF_CT_PROTO_DCCP=y<br>
CONFIG_NF_CT_PROTO_GRE=y<br>
CONFIG_NF_CT_PROTO_SCTP=y<br>
CONFIG_NF_CT_PROTO_UDPLITE=y<br>
CONFIG_NF_CONNTRACK_AMANDA=y<br>
CONFIG_NF_CONNTRACK_FTP=y<br>
CONFIG_NF_CONNTRACK_H323=y<br>
CONFIG_NF_CONNTRACK_IRC=y<br>
CONFIG_NF_CONNTRACK_BROADCAST=y<br>
CONFIG_NF_CONNTRACK_NETBIOS_NS=y<br>
# CONFIG_NF_CONNTRACK_SNMP is not set<br>
CONFIG_NF_CONNTRACK_PPTP=y<br>
CONFIG_NF_CONNTRACK_SANE=y<br>
# CONFIG_NF_CONNTRACK_SIP is not set<br>
CONFIG_NF_CONNTRACK_TFTP=y<br>
CONFIG_NF_CT_NETLINK=y<br>
# CONFIG_NF_CT_NETLINK_TIMEOUT is not set<br>
# CONFIG_NETFILTER_NETLINK_QUEUE_CT is not set<br>
CONFIG_NF_NAT=y<br>
CONFIG_NF_NAT_NEEDED=y<br>
CONFIG_NF_NAT_PROTO_DCCP=y<br>
CONFIG_NF_NAT_PROTO_UDPLITE=y<br>
CONFIG_NF_NAT_PROTO_SCTP=y<br>
CONFIG_NF_NAT_AMANDA=y<br>
CONFIG_NF_NAT_FTP=y<br>
CONFIG_NF_NAT_IRC=y<br>
# CONFIG_NF_NAT_SIP is not set<br>
CONFIG_NF_NAT_TFTP=y<br>
# CONFIG_NF_TABLES is not set<br>
CONFIG_NETFILTER_XTABLES=y<br>
<br>
#<br>
# Xtables combined modules<br>
#<br>
CONFIG_NETFILTER_XT_MARK=y<br>
CONFIG_NETFILTER_XT_CONNMARK=y<br>
<br>
#<br>
# Xtables targets<br>
#<br>
# CONFIG_NETFILTER_XT_TARGET_AUDIT is not set<br>
CONFIG_NETFILTER_XT_TARGET_CHECKSUM=y<br>
CONFIG_NETFILTER_XT_TARGET_CLASSIFY=y<br>
CONFIG_NETFILTER_XT_TARGET_CONNMARK=y<br>
CONFIG_NETFILTER_XT_TARGET_CONNSECMARK=y<br>
CONFIG_NETFILTER_XT_TARGET_CT=y<br>
# CONFIG_NETFILTER_XT_TARGET_DSCP is not set<br>
# CONFIG_NETFILTER_XT_TARGET_HL is not set<br>
# CONFIG_NETFILTER_XT_TARGET_HMARK is not set<br>
CONFIG_NETFILTER_XT_TARGET_IDLETIMER=y<br>
CONFIG_NETFILTER_XT_TARGET_HARDIDLETIMER=y<br>
# CONFIG_NETFILTER_XT_TARGET_LED is not set<br>
CONFIG_NETFILTER_XT_TARGET_LOG=y<br>
CONFIG_NETFILTER_XT_TARGET_MARK=y<br>
CONFIG_NETFILTER_XT_NAT=y<br>
CONFIG_NETFILTER_XT_TARGET_NETMAP=y<br>
CONFIG_NETFILTER_XT_TARGET_NFLOG=y<br>
CONFIG_NETFILTER_XT_TARGET_NFQUEUE=y<br>
CONFIG_NETFILTER_XT_TARGET_NOTRACK=y<br>
# CONFIG_NETFILTER_XT_TARGET_RATEEST is not set<br>
CONFIG_NETFILTER_XT_TARGET_REDIRECT=y<br>
CONFIG_NETFILTER_XT_TARGET_TEE=y<br>
CONFIG_NETFILTER_XT_TARGET_TPROXY=y<br>
CONFIG_NETFILTER_XT_TARGET_TRACE=y<br>
CONFIG_NETFILTER_XT_TARGET_SECMARK=y<br>
CONFIG_NETFILTER_XT_TARGET_TCPMSS=y<br>
# CONFIG_NETFILTER_XT_TARGET_TCPOPTSTRIP is not set<br>
<br>
#<br>
# Xtables matches<br>
#<br>
# CONFIG_NETFILTER_XT_MATCH_ADDRTYPE is not set<br>
# CONFIG_NETFILTER_XT_MATCH_BPF is not set<br>
# CONFIG_NETFILTER_XT_MATCH_CGROUP is not set<br>
# CONFIG_NETFILTER_XT_MATCH_CLUSTER is not set<br>
CONFIG_NETFILTER_XT_MATCH_COMMENT=y<br>
# CONFIG_NETFILTER_XT_MATCH_CONNBYTES is not set<br>
# CONFIG_NETFILTER_XT_MATCH_CONNLABEL is not set<br>
CONFIG_NETFILTER_XT_MATCH_CONNLIMIT=y<br>
CONFIG_NETFILTER_XT_MATCH_CONNMARK=y<br>
CONFIG_NETFILTER_XT_MATCH_CONNTRACK=y<br>
# CONFIG_NETFILTER_XT_MATCH_CPU is not set<br>
# CONFIG_NETFILTER_XT_MATCH_DCCP is not set<br>
# CONFIG_NETFILTER_XT_MATCH_DEVGROUP is not set<br>
CONFIG_NETFILTER_XT_MATCH_DSCP=y<br>
CONFIG_NETFILTER_XT_MATCH_ECN=y<br>
CONFIG_NETFILTER_XT_MATCH_ESP=y<br>
CONFIG_NETFILTER_XT_MATCH_HASHLIMIT=y<br>
CONFIG_NETFILTER_XT_MATCH_HELPER=y<br>
CONFIG_NETFILTER_XT_MATCH_HL=y<br>
# CONFIG_NETFILTER_XT_MATCH_IPCOMP is not set<br>
CONFIG_NETFILTER_XT_MATCH_IPRANGE=y<br>
CONFIG_NETFILTER_XT_MATCH_L2TP=y<br>
CONFIG_NETFILTER_XT_MATCH_LENGTH=y<br>
CONFIG_NETFILTER_XT_MATCH_LIMIT=y<br>
CONFIG_NETFILTER_XT_MATCH_MAC=y<br>
CONFIG_NETFILTER_XT_MATCH_MARK=y<br>
CONFIG_NETFILTER_XT_MATCH_MULTIPORT=y<br>
CONFIG_NETFILTER_XT_MATCH_NFACCT=m<br>
# CONFIG_NETFILTER_XT_MATCH_OSF is not set<br>
# CONFIG_NETFILTER_XT_MATCH_OWNER is not set<br>
CONFIG_NETFILTER_XT_MATCH_POLICY=y<br>
# CONFIG_NETFILTER_XT_MATCH_PHYSDEV is not set<br>
CONFIG_NETFILTER_XT_MATCH_PKTTYPE=y<br>
CONFIG_NETFILTER_XT_MATCH_QTAGUID=y<br>
CONFIG_NETFILTER_XT_MATCH_QUOTA=y<br>
CONFIG_NETFILTER_XT_MATCH_QUOTA2=y<br>
# CONFIG_NETFILTER_XT_MATCH_RATEEST is not set<br>
# CONFIG_NETFILTER_XT_MATCH_REALM is not set<br>
# CONFIG_NETFILTER_XT_MATCH_RECENT is not set<br>
# CONFIG_NETFILTER_XT_MATCH_SCTP is not set<br>
CONFIG_NETFILTER_XT_MATCH_SOCKET=y<br>
CONFIG_NETFILTER_XT_MATCH_STATE=y<br>
CONFIG_NETFILTER_XT_MATCH_STATISTIC=y<br>
CONFIG_NETFILTER_XT_MATCH_STRING=y<br>
# CONFIG_NETFILTER_XT_MATCH_TCPMSS is not set<br>
CONFIG_NETFILTER_XT_MATCH_TIME=y<br>
CONFIG_NETFILTER_XT_MATCH_U32=y<br>
# CONFIG_IP_SET is not set<br>
# CONFIG_IP_VS is not set<br>
<br>
#<br>
# IP: Netfilter Configuration<br>
#<br>
CONFIG_NF_DEFRAG_IPV4=y<br>
CONFIG_NF_CONNTRACK_IPV4=y<br>
CONFIG_NF_CONNTRACK_PROC_COMPAT=y<br>
# CONFIG_NF_LOG_ARP is not set<br>
CONFIG_NF_LOG_IPV4=y<br>
CONFIG_NF_REJECT_IPV4=y<br>
CONFIG_NF_NAT_IPV4=y<br>
CONFIG_NF_NAT_MASQUERADE_IPV4=y<br>
CONFIG_NF_NAT_PROTO_GRE=y<br>
CONFIG_NF_NAT_PPTP=y<br>
CONFIG_NF_NAT_H323=y<br>
CONFIG_IP_NF_IPTABLES=y<br>
CONFIG_IP_NF_MATCH_AH=y<br>
CONFIG_IP_NF_MATCH_ECN=y<br>
# CONFIG_IP_NF_MATCH_RPFILTER is not set<br>
CONFIG_IP_NF_MATCH_TTL=y<br>
CONFIG_IP_NF_FILTER=y<br>
CONFIG_IP_NF_TARGET_REJECT=y<br>
# CONFIG_IP_NF_TARGET_SYNPROXY is not set<br>
CONFIG_IP_NF_NAT=y<br>
CONFIG_IP_NF_TARGET_MASQUERADE=y<br>
CONFIG_IP_NF_TARGET_NATTYPE_MODULE=y<br>
CONFIG_IP_NF_TARGET_NETMAP=y<br>
CONFIG_IP_NF_TARGET_REDIRECT=y<br>
CONFIG_IP_NF_MANGLE=y<br>
# CONFIG_IP_NF_TARGET_CLUSTERIP is not set<br>
# CONFIG_IP_NF_TARGET_ECN is not set<br>
# CONFIG_IP_NF_TARGET_TTL is not set<br>
CONFIG_IP_NF_RAW=y<br>
CONFIG_IP_NF_SECURITY=y<br>
CONFIG_IP_NF_ARPTABLES=y<br>
CONFIG_IP_NF_ARPFILTER=y<br>
CONFIG_IP_NF_ARP_MANGLE=y<br>
<br>
#<br>
# IPv6: Netfilter Configuration<br>
#<br>
CONFIG_NF_DEFRAG_IPV6=y<br>
CONFIG_NF_CONNTRACK_IPV6=y<br>
CONFIG_NF_REJECT_IPV6=y<br>
CONFIG_NF_LOG_IPV6=y<br>
CONFIG_NF_NAT_IPV6=y<br>
# CONFIG_NF_NAT_MASQUERADE_IPV6 is not set<br>
CONFIG_IP6_NF_IPTABLES=y<br>
# CONFIG_IP6_NF_MATCH_AH is not set<br>
# CONFIG_IP6_NF_MATCH_EUI64 is not set<br>
# CONFIG_IP6_NF_MATCH_FRAG is not set<br>
# CONFIG_IP6_NF_MATCH_OPTS is not set<br>
# CONFIG_IP6_NF_MATCH_HL is not set<br>
# CONFIG_IP6_NF_MATCH_IPV6HEADER is not set<br>
# CONFIG_IP6_NF_MATCH_MH is not set<br>
CONFIG_IP6_NF_MATCH_RPFILTER=y<br>
# CONFIG_IP6_NF_MATCH_RT is not set<br>
# CONFIG_IP6_NF_TARGET_HL is not set<br>
CONFIG_IP6_NF_FILTER=y<br>
CONFIG_IP6_NF_TARGET_REJECT=y<br>
# CONFIG_IP6_NF_TARGET_SYNPROXY is not set<br>
CONFIG_IP6_NF_MANGLE=y<br>
CONFIG_IP6_NF_RAW=y<br>
# CONFIG_IP6_NF_SECURITY is not set<br>
# CONFIG_IP6_NF_NAT is not set<br>
CONFIG_BRIDGE_NF_EBTABLES=y<br>
CONFIG_BRIDGE_EBT_BROUTE=y<br>
# CONFIG_BRIDGE_EBT_T_FILTER is not set<br>
# CONFIG_BRIDGE_EBT_T_NAT is not set<br>
# CONFIG_BRIDGE_EBT_802_3 is not set<br>
# CONFIG_BRIDGE_EBT_AMONG is not set<br>
# CONFIG_BRIDGE_EBT_ARP is not set<br>
# CONFIG_BRIDGE_EBT_IP is not set<br>
# CONFIG_BRIDGE_EBT_IP6 is not set<br>
# CONFIG_BRIDGE_EBT_LIMIT is not set<br>
# CONFIG_BRIDGE_EBT_MARK is not set<br>
# CONFIG_BRIDGE_EBT_PKTTYPE is not set<br>
# CONFIG_BRIDGE_EBT_STP is not set<br>
# CONFIG_BRIDGE_EBT_VLAN is not set<br>
# CONFIG_BRIDGE_EBT_ARPREPLY is not set<br>
# CONFIG_BRIDGE_EBT_DNAT is not set<br>
# CONFIG_BRIDGE_EBT_MARK_T is not set<br>
# CONFIG_BRIDGE_EBT_REDIRECT is not set<br>
# CONFIG_BRIDGE_EBT_SNAT is not set<br>
# CONFIG_BRIDGE_EBT_LOG is not set<br>
# CONFIG_BRIDGE_EBT_NFLOG is not set<br>
# CONFIG_IP_DCCP is not set<br>
# CONFIG_IP_SCTP is not set<br>
# CONFIG_RDS is not set<br>
# CONFIG_TIPC is not set<br>
# CONFIG_ATM is not set<br>
CONFIG_L2TP=y<br>
CONFIG_L2TP_DEBUGFS=y<br>
CONFIG_L2TP_V3=y<br>
CONFIG_L2TP_IP=y<br>
CONFIG_L2TP_ETH=y<br>
CONFIG_STP=y<br>
CONFIG_BRIDGE=y<br>
CONFIG_BRIDGE_IGMP_SNOOPING=y<br>
# CONFIG_BRIDGE_VLAN_FILTERING is not set<br>
CONFIG_HAVE_NET_DSA=y<br>
CONFIG_VLAN_8021Q=y<br>
# CONFIG_VLAN_8021Q_GVRP is not set<br>
# CONFIG_VLAN_8021Q_MVRP is not set<br>
# CONFIG_DECNET is not set<br>
CONFIG_LLC=y<br>
# CONFIG_LLC2 is not set<br>
# CONFIG_IPX is not set<br>
# CONFIG_ATALK is not set<br>
# CONFIG_X25 is not set<br>
# CONFIG_LAPB is not set<br>
# CONFIG_PHONET is not set<br>
# CONFIG_6LOWPAN is not set<br>
# CONFIG_IEEE802154 is not set<br>
CONFIG_NET_SCHED=y<br>
<br>
#<br>
# Queueing/Scheduling<br>
#<br>
# CONFIG_NET_SCH_CBQ is not set<br>
CONFIG_NET_SCH_HTB=y<br>
# CONFIG_NET_SCH_HFSC is not set<br>
CONFIG_NET_SCH_PRIO=y<br>
# CONFIG_NET_SCH_MULTIQ is not set<br>
# CONFIG_NET_SCH_RED is not set<br>
# CONFIG_NET_SCH_SFB is not set<br>
# CONFIG_NET_SCH_SFQ is not set<br>
# CONFIG_NET_SCH_TEQL is not set<br>
# CONFIG_NET_SCH_TBF is not set<br>
# CONFIG_NET_SCH_GRED is not set<br>
# CONFIG_NET_SCH_DSMARK is not set<br>
# CONFIG_NET_SCH_NETEM is not set<br>
# CONFIG_NET_SCH_DRR is not set<br>
# CONFIG_NET_SCH_MQPRIO is not set<br>
# CONFIG_NET_SCH_CHOKE is not set<br>
# CONFIG_NET_SCH_QFQ is not set<br>
# CONFIG_NET_SCH_CODEL is not set<br>
# CONFIG_NET_SCH_FQ_CODEL is not set<br>
# CONFIG_NET_SCH_FQ is not set<br>
# CONFIG_NET_SCH_HHF is not set<br>
# CONFIG_NET_SCH_PIE is not set<br>
# CONFIG_NET_SCH_INGRESS is not set<br>
# CONFIG_NET_SCH_PLUG is not set<br>
<br>
#<br>
# Classification<br>
#<br>
CONFIG_NET_CLS=y<br>
# CONFIG_NET_CLS_BASIC is not set<br>
# CONFIG_NET_CLS_TCINDEX is not set<br>
# CONFIG_NET_CLS_ROUTE4 is not set<br>
CONFIG_NET_CLS_FW=y<br>
CONFIG_NET_CLS_U32=y<br>
# CONFIG_CLS_U32_PERF is not set<br>
CONFIG_CLS_U32_MARK=y<br>
# CONFIG_NET_CLS_RSVP is not set<br>
# CONFIG_NET_CLS_RSVP6 is not set<br>
CONFIG_NET_CLS_FLOW=y<br>
CONFIG_NET_CLS_CGROUP=y<br>
# CONFIG_NET_CLS_BPF is not set<br>
CONFIG_NET_EMATCH=y<br>
CONFIG_NET_EMATCH_STACK=32<br>
CONFIG_NET_EMATCH_CMP=y<br>
CONFIG_NET_EMATCH_NBYTE=y<br>
CONFIG_NET_EMATCH_U32=y<br>
CONFIG_NET_EMATCH_META=y<br>
CONFIG_NET_EMATCH_TEXT=y<br>
CONFIG_NET_CLS_ACT=y<br>
# CONFIG_NET_ACT_POLICE is not set<br>
# CONFIG_NET_ACT_GACT is not set<br>
# CONFIG_NET_ACT_MIRRED is not set<br>
# CONFIG_NET_ACT_IPT is not set<br>
# CONFIG_NET_ACT_NAT is not set<br>
# CONFIG_NET_ACT_PEDIT is not set<br>
# CONFIG_NET_ACT_SIMP is not set<br>
# CONFIG_NET_ACT_SKBEDIT is not set<br>
# CONFIG_NET_ACT_CSUM is not set<br>
# CONFIG_NET_CLS_IND is not set<br>
CONFIG_NET_SCH_FIFO=y<br>
# CONFIG_DCB is not set<br>
# CONFIG_DNS_RESOLVER is not set<br>
# CONFIG_BATMAN_ADV is not set<br>
# CONFIG_OPENVSWITCH is not set<br>
# CONFIG_VSOCKETS is not set<br>
CONFIG_NETLINK_DIAG=y<br>
# CONFIG_NET_MPLS_GSO is not set<br>
# CONFIG_HSR is not set<br>
CONFIG_RMNET_DATA=y<br>
CONFIG_RMNET_DATA_FC=y<br>
CONFIG_RMNET_DATA_DEBUG_PKT=y<br>
CONFIG_RPS=y<br>
CONFIG_RFS_ACCEL=y<br>
CONFIG_XPS=y<br>
# CONFIG_CGROUP_NET_PRIO is not set<br>
CONFIG_CGROUP_NET_CLASSID=y<br>
CONFIG_NET_RX_BUSY_POLL=y<br>
CONFIG_BQL=y<br>
# CONFIG_BPF_JIT is not set<br>
CONFIG_NET_FLOW_LIMIT=y<br>
CONFIG_SOCKEV_NLMCAST=y<br>
<br>
#<br>
# Network testing<br>
#<br>
# CONFIG_NET_PKTGEN is not set<br>
# CONFIG_NET_DROP_MONITOR is not set<br>
# CONFIG_HAMRADIO is not set<br>
# CONFIG_CAN is not set<br>
# CONFIG_IRDA is not set<br>
CONFIG_BT=m<br>
CONFIG_BT_RFCOMM=m<br>
CONFIG_BT_RFCOMM_TTY=y<br>
CONFIG_BT_BNEP=m<br>
CONFIG_BT_BNEP_MC_FILTER=y<br>
CONFIG_BT_BNEP_PROTO_FILTER=y<br>
CONFIG_BT_HIDP=m<br>
<br>
#<br>
# Bluetooth device drivers<br>
#<br>
CONFIG_BT_HCISMD=m<br>
# CONFIG_BT_HCIBTUSB is not set<br>
# CONFIG_BT_HCIBTSDIO is not set<br>
CONFIG_BT_HCIUART=m<br>
CONFIG_BT_HCIUART_H4=y<br>
# CONFIG_BT_HCIUART_BCSP is not set<br>
# CONFIG_BT_HCIUART_ATH3K is not set<br>
# CONFIG_BT_HCIUART_LL is not set<br>
# CONFIG_BT_HCIUART_3WIRE is not set<br>
# CONFIG_BT_HCIUART_IBS is not set<br>
# CONFIG_BT_HCIBCM203X is not set<br>
# CONFIG_BT_HCIBPA10X is not set<br>
# CONFIG_BT_HCIBFUSB is not set<br>
# CONFIG_BT_HCIVHCI is not set<br>
# CONFIG_BT_MRVL is not set<br>
CONFIG_MSM_BT_POWER=m<br>
# CONFIG_BTFM_SLIM is not set<br>
# CONFIG_AF_RXRPC is not set<br>
CONFIG_FIB_RULES=y<br>
CONFIG_WIRELESS=y<br>
CONFIG_WIRELESS_EXT=y<br>
CONFIG_WEXT_CORE=y<br>
CONFIG_WEXT_PROC=y<br>
CONFIG_WEXT_SPY=y<br>
CONFIG_WEXT_PRIV=y<br>
CONFIG_CFG80211=y<br>
CONFIG_NL80211_TESTMODE=y<br>
# CONFIG_CFG80211_DEVELOPER_WARNINGS is not set<br>
# CONFIG_CFG80211_REG_DEBUG is not set<br>
# CONFIG_CFG80211_CERTIFICATION_ONUS is not set<br>
CONFIG_CFG80211_DEFAULT_PS=y<br>
# CONFIG_CFG80211_DEBUGFS is not set<br>
CONFIG_CFG80211_INTERNAL_REGDB=y<br>
# CONFIG_CFG80211_WEXT is not set<br>
# CONFIG_LIB80211 is not set<br>
# CONFIG_MAC80211 is not set<br>
# CONFIG_WIMAX is not set<br>
CONFIG_RFKILL=y<br>
CONFIG_RFKILL_PM=y<br>
CONFIG_RFKILL_LEDS=y<br>
# CONFIG_RFKILL_INPUT is not set<br>
# CONFIG_RFKILL_REGULATOR is not set<br>
# CONFIG_RFKILL_GPIO is not set<br>
# CONFIG_NET_9P is not set<br>
# CONFIG_CAIF is not set<br>
# CONFIG_CEPH_LIB is not set<br>
# CONFIG_NFC is not set<br>
CONFIG_NFC_NQ=y<br>
CONFIG_IPC_ROUTER=y<br>
CONFIG_IPC_ROUTER_SECURITY=y<br>
CONFIG_HAVE_BPF_JIT=y<br>
<br>
#<br>
# Device Drivers<br>
#<br>
<br>
#<br>
# Generic Driver Options<br>
#<br>
CONFIG_UEVENT_HELPER=y<br>
CONFIG_UEVENT_HELPER_PATH=""<br>
CONFIG_DEVTMPFS=y<br>
CONFIG_DEVTMPFS_MOUNT=y<br>
CONFIG_STANDALONE=y<br>
CONFIG_PREVENT_FIRMWARE_BUILD=y<br>
CONFIG_FW_LOADER=y<br>
CONFIG_FIRMWARE_IN_KERNEL=y<br>
CONFIG_EXTRA_FIRMWARE=""<br>
CONFIG_FW_LOADER_USER_HELPER=y<br>
CONFIG_FW_LOADER_USER_HELPER_FALLBACK=y<br>
# CONFIG_FW_CACHE is not set<br>
CONFIG_WANT_DEV_COREDUMP=y<br>
CONFIG_ALLOW_DEV_COREDUMP=y<br>
CONFIG_DEV_COREDUMP=y<br>
# CONFIG_DEBUG_DRIVER is not set<br>
# CONFIG_DEBUG_DEVRES is not set<br>
# CONFIG_SYS_HYPERVISOR is not set<br>
# CONFIG_GENERIC_CPU_DEVICES is not set<br>
CONFIG_HAVE_CPU_AUTOPROBE=y<br>
CONFIG_GENERIC_CPU_AUTOPROBE=y<br>
CONFIG_SOC_BUS=y<br>
CONFIG_REGMAP=y<br>
CONFIG_REGMAP_I2C=y<br>
CONFIG_REGMAP_SPI=y<br>
CONFIG_REGMAP_SWR=y<br>
CONFIG_REGMAP_ALLOW_WRITE_DEBUGFS=y<br>
CONFIG_DMA_SHARED_BUFFER=y<br>
# CONFIG_FENCE_TRACE is not set<br>
CONFIG_DMA_CMA=y<br>
<br>
#<br>
# Default contiguous memory area size:<br>
#<br>
CONFIG_CMA_SIZE_MBYTES=16<br>
CONFIG_CMA_SIZE_SEL_MBYTES=y<br>
# CONFIG_CMA_SIZE_SEL_PERCENTAGE is not set<br>
# CONFIG_CMA_SIZE_SEL_MIN is not set<br>
# CONFIG_CMA_SIZE_SEL_MAX is not set<br>
CONFIG_CMA_ALIGNMENT=8<br>
<br>
#<br>
# Bus devices<br>
#<br>
# CONFIG_ARM_CCN is not set<br>
# CONFIG_VEXPRESS_CONFIG is not set<br>
# CONFIG_CONNECTOR is not set<br>
# CONFIG_MTD is not set<br>
CONFIG_DTC=y<br>
CONFIG_OF=y<br>
<br>
#<br>
# Device Tree and Open Firmware support<br>
#<br>
# CONFIG_OF_UNITTEST is not set<br>
CONFIG_OF_FLATTREE=y<br>
CONFIG_OF_EARLY_FLATTREE=y<br>
CONFIG_OF_ADDRESS=y<br>
CONFIG_OF_ADDRESS_PCI=y<br>
CONFIG_OF_IRQ=y<br>
CONFIG_OF_NET=y<br>
CONFIG_OF_MDIO=y<br>
CONFIG_OF_PCI=y<br>
CONFIG_OF_PCI_IRQ=y<br>
CONFIG_OF_SPMI=y<br>
CONFIG_OF_RESERVED_MEM=y<br>
CONFIG_OF_SLIMBUS=y<br>
CONFIG_OF_CORESIGHT=y<br>
CONFIG_OF_BATTERYDATA=y<br>
# CONFIG_OF_OVERLAY is not set<br>
# CONFIG_PARPORT is not set<br>
CONFIG_BLK_DEV=y<br>
# CONFIG_BLK_DEV_NULL_BLK is not set<br>
# CONFIG_BLK_DEV_PCIESSD_MTIP32XX is not set<br>
CONFIG_ZRAM=y<br>
CONFIG_ZRAM_LZ4_COMPRESS=y<br>
# CONFIG_BLK_CPQ_CISS_DA is not set<br>
# CONFIG_BLK_DEV_DAC960 is not set<br>
# CONFIG_BLK_DEV_UMEM is not set<br>
# CONFIG_BLK_DEV_COW_COMMON is not set<br>
CONFIG_BLK_DEV_LOOP=y<br>
CONFIG_BLK_DEV_LOOP_MIN_COUNT=8<br>
# CONFIG_BLK_DEV_CRYPTOLOOP is not set<br>
# CONFIG_BLK_DEV_DRBD is not set<br>
# CONFIG_BLK_DEV_NBD is not set<br>
# CONFIG_BLK_DEV_NVME is not set<br>
# CONFIG_BLK_DEV_SKD is not set<br>
# CONFIG_BLK_DEV_SX8 is not set<br>
CONFIG_BLK_DEV_RAM=y<br>
CONFIG_BLK_DEV_RAM_COUNT=16<br>
CONFIG_BLK_DEV_RAM_SIZE=8192<br>
# CONFIG_BLK_DEV_XIP is not set<br>
# CONFIG_CDROM_PKTCDVD is not set<br>
# CONFIG_ATA_OVER_ETH is not set<br>
# CONFIG_BLK_DEV_RBD is not set<br>
# CONFIG_BLK_DEV_RSXX is not set<br>
<br>
#<br>
# Misc devices<br>
#<br>
# CONFIG_SENSORS_LIS3LV02D is not set<br>
# CONFIG_AD525X_DPOT is not set<br>
# CONFIG_DUMMY_IRQ is not set<br>
# CONFIG_PHANTOM is not set<br>
# CONFIG_SGI_IOC4 is not set<br>
# CONFIG_TIFM_CORE is not set<br>
# CONFIG_ICS932S401 is not set<br>
# CONFIG_ENCLOSURE_SERVICES is not set<br>
# CONFIG_HP_ILO is not set<br>
# CONFIG_APDS9802ALS is not set<br>
# CONFIG_ISL29003 is not set<br>
# CONFIG_ISL29020 is not set<br>
# CONFIG_SENSORS_TSL2550 is not set<br>
# CONFIG_SENSORS_BH1780 is not set<br>
# CONFIG_SENSORS_BH1770 is not set<br>
# CONFIG_SENSORS_APDS990X is not set<br>
# CONFIG_HMC6352 is not set<br>
# CONFIG_DS1682 is not set<br>
# CONFIG_TI_DAC7512 is not set<br>
CONFIG_UID_STAT=y<br>
# CONFIG_BMP085_I2C is not set<br>
# CONFIG_BMP085_SPI is not set<br>
# CONFIG_USB_SWITCH_FSA9480 is not set<br>
# CONFIG_LATTICE_ECP3_CONFIG is not set<br>
# CONFIG_SRAM is not set<br>
CONFIG_QSEECOM=y<br>
CONFIG_HDCP_QSEECOM=y<br>
# CONFIG_UID_SYS_STATS is not set<br>
CONFIG_USB_EXT_TYPE_C_PERICOM=y<br>
# CONFIG_USB_EXT_TYPE_C_TI is not set<br>
# CONFIG_TI_DRV2667 is not set<br>
# CONFIG_QPNP_MISC is not set<br>
CONFIG_FORCE_FAST_CHARGE=y<br>
# CONFIG_C2PORT is not set<br>
<br>
#<br>
# EEPROM support<br>
#<br>
# CONFIG_EEPROM_AT24 is not set<br>
# CONFIG_EEPROM_AT25 is not set<br>
# CONFIG_EEPROM_LEGACY is not set<br>
# CONFIG_EEPROM_MAX6875 is not set<br>
# CONFIG_EEPROM_93CX6 is not set<br>
# CONFIG_EEPROM_93XX46 is not set<br>
# CONFIG_CB710_CORE is not set<br>
<br>
#<br>
# Texas Instruments shared transport line discipline<br>
#<br>
# CONFIG_TI_ST is not set<br>
# CONFIG_SENSORS_LIS3_SPI is not set<br>
# CONFIG_SENSORS_LIS3_I2C is not set<br>
<br>
#<br>
# Altera FPGA firmware download module<br>
#<br>
# CONFIG_ALTERA_STAPL is not set<br>
CONFIG_MSM_QDSP6V2_CODECS=y<br>
CONFIG_MSM_ULTRASOUND=y<br>
<br>
#<br>
# Intel MIC Bus Driver<br>
#<br>
<br>
#<br>
# Intel MIC Host Driver<br>
#<br>
<br>
#<br>
# Intel MIC Card Driver<br>
#<br>
# CONFIG_GENWQE is not set<br>
# CONFIG_ECHO is not set<br>
# CONFIG_CXL_BASE is not set<br>
<br>
#<br>
# SCSI device support<br>
#<br>
CONFIG_SCSI_MOD=y<br>
# CONFIG_RAID_ATTRS is not set<br>
CONFIG_SCSI=y<br>
CONFIG_SCSI_DMA=y<br>
# CONFIG_SCSI_NETLINK is not set<br>
# CONFIG_SCSI_MQ_DEFAULT is not set<br>
CONFIG_SCSI_PROC_FS=y<br>
<br>
#<br>
# SCSI support type (disk, tape, CD-ROM)<br>
#<br>
CONFIG_BLK_DEV_SD=y<br>
# CONFIG_CHR_DEV_ST is not set<br>
# CONFIG_CHR_DEV_OSST is not set<br>
# CONFIG_BLK_DEV_SR is not set<br>
CONFIG_CHR_DEV_SG=y<br>
CONFIG_CHR_DEV_SCH=y<br>
CONFIG_SCSI_CONSTANTS=y<br>
CONFIG_SCSI_LOGGING=y<br>
CONFIG_SCSI_SCAN_ASYNC=y<br>
<br>
#<br>
# SCSI Transports<br>
#<br>
# CONFIG_SCSI_SPI_ATTRS is not set<br>
# CONFIG_SCSI_FC_ATTRS is not set<br>
# CONFIG_SCSI_ISCSI_ATTRS is not set<br>
# CONFIG_SCSI_SAS_ATTRS is not set<br>
# CONFIG_SCSI_SAS_LIBSAS is not set<br>
# CONFIG_SCSI_SRP_ATTRS is not set<br>
CONFIG_SCSI_LOWLEVEL=y<br>
# CONFIG_ISCSI_TCP is not set<br>
# CONFIG_ISCSI_BOOT_SYSFS is not set<br>
# CONFIG_SCSI_CXGB3_ISCSI is not set<br>
# CONFIG_SCSI_CXGB4_ISCSI is not set<br>
# CONFIG_SCSI_BNX2_ISCSI is not set<br>
# CONFIG_BE2ISCSI is not set<br>
# CONFIG_BLK_DEV_3W_XXXX_RAID is not set<br>
# CONFIG_SCSI_HPSA is not set<br>
# CONFIG_SCSI_3W_9XXX is not set<br>
# CONFIG_SCSI_3W_SAS is not set<br>
# CONFIG_SCSI_ACARD is not set<br>
# CONFIG_SCSI_AACRAID is not set<br>
# CONFIG_SCSI_AIC7XXX is not set<br>
# CONFIG_SCSI_AIC79XX is not set<br>
# CONFIG_SCSI_AIC94XX is not set<br>
# CONFIG_SCSI_MVSAS is not set<br>
# CONFIG_SCSI_MVUMI is not set<br>
# CONFIG_SCSI_ARCMSR is not set<br>
# CONFIG_SCSI_ESAS2R is not set<br>
# CONFIG_MEGARAID_NEWGEN is not set<br>
# CONFIG_MEGARAID_LEGACY is not set<br>
# CONFIG_MEGARAID_SAS is not set<br>
# CONFIG_SCSI_MPT2SAS is not set<br>
# CONFIG_SCSI_MPT3SAS is not set<br>
CONFIG_SCSI_UFSHCD=y<br>
# CONFIG_SCSI_UFSHCD_PCI is not set<br>
CONFIG_SCSI_UFSHCD_PLATFORM=y<br>
CONFIG_SCSI_UFS_QCOM=y<br>
CONFIG_SCSI_UFS_QCOM_ICE=y<br>
# CONFIG_SCSI_HPTIOP is not set<br>
# CONFIG_SCSI_DMX3191D is not set<br>
# CONFIG_SCSI_FUTURE_DOMAIN is not set<br>
# CONFIG_SCSI_IPS is not set<br>
# CONFIG_SCSI_INITIO is not set<br>
# CONFIG_SCSI_INIA100 is not set<br>
# CONFIG_SCSI_STEX is not set<br>
# CONFIG_SCSI_SYM53C8XX_2 is not set<br>
# CONFIG_SCSI_QLOGIC_1280 is not set<br>
# CONFIG_SCSI_QLA_ISCSI is not set<br>
# CONFIG_SCSI_DC395x is not set<br>
# CONFIG_SCSI_DC390T is not set<br>
# CONFIG_SCSI_DEBUG is not set<br>
# CONFIG_SCSI_PMCRAID is not set<br>
# CONFIG_SCSI_PM8001 is not set<br>
# CONFIG_SCSI_LOWLEVEL_PCMCIA is not set<br>
# CONFIG_SCSI_DH is not set<br>
# CONFIG_SCSI_OSD_INITIATOR is not set<br>
CONFIG_HAVE_PATA_PLATFORM=y<br>
# CONFIG_ATA is not set<br>
CONFIG_MD=y<br>
# CONFIG_BLK_DEV_MD is not set<br>
# CONFIG_BCACHE is not set<br>
CONFIG_BLK_DEV_DM_BUILTIN=y<br>
CONFIG_BLK_DEV_DM=y<br>
# CONFIG_DM_DEBUG is not set<br>
CONFIG_DM_BUFIO=y<br>
CONFIG_DM_CRYPT=y<br>
CONFIG_DM_REQ_CRYPT=y<br>
# CONFIG_DM_SNAPSHOT is not set<br>
# CONFIG_DM_THIN_PROVISIONING is not set<br>
# CONFIG_DM_CACHE is not set<br>
# CONFIG_DM_ERA is not set<br>
# CONFIG_DM_MIRROR is not set<br>
# CONFIG_DM_RAID is not set<br>
# CONFIG_DM_ZERO is not set<br>
# CONFIG_DM_MULTIPATH is not set<br>
# CONFIG_DM_DELAY is not set<br>
CONFIG_DM_UEVENT=y<br>
# CONFIG_DM_FLAKEY is not set<br>
CONFIG_DM_VERITY=y<br>
# CONFIG_DM_ANDROID_VERITY is not set<br>
CONFIG_DM_VERITY_FEC=y<br>
# CONFIG_DM_SWITCH is not set<br>
# CONFIG_DM_LOG_WRITES is not set<br>
# CONFIG_TARGET_CORE is not set<br>
# CONFIG_FUSION is not set<br>
<br>
#<br>
# IEEE 1394 (FireWire) support<br>
#<br>
# CONFIG_FIREWIRE is not set<br>
# CONFIG_FIREWIRE_NOSY is not set<br>
# CONFIG_I2O is not set<br>
CONFIG_NETDEVICES=y<br>
CONFIG_MII=y<br>
CONFIG_NET_CORE=y<br>
# CONFIG_BONDING is not set<br>
# CONFIG_DUMMY is not set<br>
# CONFIG_EQUALIZER is not set<br>
# CONFIG_NET_FC is not set<br>
# CONFIG_IFB is not set<br>
# CONFIG_NET_TEAM is not set<br>
CONFIG_MACVLAN=y<br>
# CONFIG_MACVTAP is not set<br>
# CONFIG_VXLAN is not set<br>
# CONFIG_NETCONSOLE is not set<br>
# CONFIG_NETPOLL is not set<br>
# CONFIG_NET_POLL_CONTROLLER is not set<br>
CONFIG_TUN=y<br>
CONFIG_VETH=y<br>
# CONFIG_NLMON is not set<br>
# CONFIG_ARCNET is not set<br>
<br>
#<br>
# CAIF transport drivers<br>
#<br>
<br>
#<br>
# Distributed Switch Architecture drivers<br>
#<br>
# CONFIG_NET_DSA_MV88E6XXX is not set<br>
# CONFIG_NET_DSA_MV88E6060 is not set<br>
# CONFIG_NET_DSA_MV88E6XXX_NEED_PPU is not set<br>
# CONFIG_NET_DSA_MV88E6131 is not set<br>
# CONFIG_NET_DSA_MV88E6123_61_65 is not set<br>
# CONFIG_NET_DSA_MV88E6171 is not set<br>
# CONFIG_NET_DSA_BCM_SF2 is not set<br>
CONFIG_ETHERNET=y<br>
CONFIG_NET_VENDOR_3COM=y<br>
# CONFIG_VORTEX is not set<br>
# CONFIG_TYPHOON is not set<br>
CONFIG_NET_VENDOR_ADAPTEC=y<br>
# CONFIG_ADAPTEC_STARFIRE is not set<br>
CONFIG_NET_VENDOR_AGERE=y<br>
# CONFIG_ET131X is not set<br>
CONFIG_NET_VENDOR_ALTEON=y<br>
# CONFIG_ACENIC is not set<br>
# CONFIG_ALTERA_TSE is not set<br>
CONFIG_NET_VENDOR_AMD=y<br>
# CONFIG_AMD8111_ETH is not set<br>
# CONFIG_PCNET32 is not set<br>
# CONFIG_AMD_XGBE is not set<br>
# CONFIG_NET_XGENE is not set<br>
CONFIG_NET_VENDOR_ARC=y<br>
# CONFIG_ARC_EMAC is not set<br>
# CONFIG_EMAC_ROCKCHIP is not set<br>
CONFIG_NET_VENDOR_ATHEROS=y<br>
# CONFIG_ATL2 is not set<br>
# CONFIG_ATL1 is not set<br>
# CONFIG_ATL1E is not set<br>
# CONFIG_ATL1C is not set<br>
# CONFIG_ALX is not set<br>
CONFIG_NET_VENDOR_BROADCOM=y<br>
# CONFIG_B44 is not set<br>
# CONFIG_BCMGENET is not set<br>
# CONFIG_BNX2 is not set<br>
# CONFIG_CNIC is not set<br>
# CONFIG_TIGON3 is not set<br>
# CONFIG_BNX2X is not set<br>
# CONFIG_SYSTEMPORT is not set<br>
CONFIG_NET_VENDOR_BROCADE=y<br>
# CONFIG_BNA is not set<br>
CONFIG_NET_VENDOR_CHELSIO=y<br>
# CONFIG_CHELSIO_T1 is not set<br>
# CONFIG_CHELSIO_T3 is not set<br>
# CONFIG_CHELSIO_T4 is not set<br>
# CONFIG_CHELSIO_T4VF is not set<br>
CONFIG_NET_VENDOR_CISCO=y<br>
# CONFIG_ENIC is not set<br>
# CONFIG_DNET is not set<br>
CONFIG_NET_VENDOR_DEC=y<br>
# CONFIG_NET_TULIP is not set<br>
CONFIG_NET_VENDOR_DLINK=y<br>
# CONFIG_DL2K is not set<br>
# CONFIG_SUNDANCE is not set<br>
CONFIG_NET_VENDOR_EMULEX=y<br>
# CONFIG_BE2NET is not set<br>
CONFIG_NET_VENDOR_EXAR=y<br>
# CONFIG_S2IO is not set<br>
# CONFIG_VXGE is not set<br>
CONFIG_NET_VENDOR_HP=y<br>
# CONFIG_HP100 is not set<br>
CONFIG_NET_VENDOR_INTEL=y<br>
# CONFIG_E100 is not set<br>
# CONFIG_E1000 is not set<br>
# CONFIG_E1000E is not set<br>
# CONFIG_IGB is not set<br>
# CONFIG_IGBVF is not set<br>
# CONFIG_IXGB is not set<br>
# CONFIG_IXGBE is not set<br>
# CONFIG_IXGBEVF is not set<br>
# CONFIG_I40E is not set<br>
# CONFIG_I40EVF is not set<br>
# CONFIG_FM10K is not set<br>
CONFIG_NET_VENDOR_I825XX=y<br>
# CONFIG_IP1000 is not set<br>
# CONFIG_JME is not set<br>
CONFIG_NET_VENDOR_MARVELL=y<br>
# CONFIG_MVMDIO is not set<br>
# CONFIG_SKGE is not set<br>
# CONFIG_SKY2 is not set<br>
CONFIG_NET_VENDOR_MELLANOX=y<br>
# CONFIG_MLX4_EN is not set<br>
# CONFIG_MLX4_CORE is not set<br>
# CONFIG_MLX5_CORE is not set<br>
CONFIG_NET_VENDOR_MICREL=y<br>
# CONFIG_KS8842 is not set<br>
# CONFIG_KS8851 is not set<br>
# CONFIG_KS8851_MLL is not set<br>
# CONFIG_KSZ884X_PCI is not set<br>
CONFIG_NET_VENDOR_MICROCHIP=y<br>
# CONFIG_ENC28J60 is not set<br>
# CONFIG_ECM_IPA is not set<br>
CONFIG_RNDIS_IPA=y<br>
# CONFIG_MSM_RMNET_BAM is not set<br>
CONFIG_NET_VENDOR_MYRI=y<br>
# CONFIG_MYRI10GE is not set<br>
# CONFIG_FEALNX is not set<br>
CONFIG_NET_VENDOR_NATSEMI=y<br>
# CONFIG_NATSEMI is not set<br>
# CONFIG_NS83820 is not set<br>
CONFIG_NET_VENDOR_8390=y<br>
# CONFIG_NE2K_PCI is not set<br>
CONFIG_NET_VENDOR_NVIDIA=y<br>
# CONFIG_FORCEDETH is not set<br>
CONFIG_NET_VENDOR_OKI=y<br>
# CONFIG_ETHOC is not set<br>
CONFIG_NET_PACKET_ENGINE=y<br>
# CONFIG_HAMACHI is not set<br>
# CONFIG_YELLOWFIN is not set<br>
CONFIG_NET_VENDOR_QLOGIC=y<br>
# CONFIG_QLA3XXX is not set<br>
# CONFIG_QLCNIC is not set<br>
# CONFIG_QLGE is not set<br>
# CONFIG_NETXEN_NIC is not set<br>
CONFIG_NET_VENDOR_QUALCOMM=y<br>
# CONFIG_QCA7000 is not set<br>
# CONFIG_QCOM_EMAC is not set<br>
CONFIG_NET_VENDOR_REALTEK=y<br>
# CONFIG_8139CP is not set<br>
# CONFIG_8139TOO is not set<br>
# CONFIG_R8169 is not set<br>
CONFIG_NET_VENDOR_RDC=y<br>
# CONFIG_R6040 is not set<br>
CONFIG_NET_VENDOR_SAMSUNG=y<br>
# CONFIG_SXGBE_ETH is not set<br>
CONFIG_NET_VENDOR_SEEQ=y<br>
CONFIG_NET_VENDOR_SILAN=y<br>
# CONFIG_SC92031 is not set<br>
CONFIG_NET_VENDOR_SIS=y<br>
# CONFIG_SIS900 is not set<br>
# CONFIG_SIS190 is not set<br>
# CONFIG_SFC is not set<br>
CONFIG_NET_VENDOR_SMSC=y<br>
# CONFIG_SMC91X is not set<br>
# CONFIG_EPIC100 is not set<br>
# CONFIG_SMSC911X is not set<br>
# CONFIG_SMSC9420 is not set<br>
CONFIG_NET_VENDOR_STMICRO=y<br>
# CONFIG_STMMAC_ETH is not set<br>
CONFIG_NET_VENDOR_SUN=y<br>
# CONFIG_HAPPYMEAL is not set<br>
# CONFIG_SUNGEM is not set<br>
# CONFIG_CASSINI is not set<br>
# CONFIG_NIU is not set<br>
CONFIG_NET_VENDOR_TEHUTI=y<br>
# CONFIG_TEHUTI is not set<br>
CONFIG_NET_VENDOR_TI=y<br>
# CONFIG_TLAN is not set<br>
CONFIG_NET_VENDOR_VIA=y<br>
# CONFIG_VIA_RHINE is not set<br>
# CONFIG_VIA_VELOCITY is not set<br>
CONFIG_NET_VENDOR_WIZNET=y<br>
# CONFIG_WIZNET_W5100 is not set<br>
# CONFIG_WIZNET_W5300 is not set<br>
# CONFIG_FDDI is not set<br>
# CONFIG_HIPPI is not set<br>
CONFIG_PHYLIB=y<br>
<br>
#<br>
# MII PHY device drivers<br>
#<br>
# CONFIG_AT803X_PHY is not set<br>
# CONFIG_AMD_PHY is not set<br>
# CONFIG_AMD_XGBE_PHY is not set<br>
# CONFIG_MARVELL_PHY is not set<br>
# CONFIG_DAVICOM_PHY is not set<br>
# CONFIG_QSEMI_PHY is not set<br>
# CONFIG_LXT_PHY is not set<br>
# CONFIG_CICADA_PHY is not set<br>
# CONFIG_VITESSE_PHY is not set<br>
# CONFIG_SMSC_PHY is not set<br>
# CONFIG_BROADCOM_PHY is not set<br>
# CONFIG_BCM7XXX_PHY is not set<br>
# CONFIG_BCM87XX_PHY is not set<br>
# CONFIG_ICPLUS_PHY is not set<br>
# CONFIG_REALTEK_PHY is not set<br>
# CONFIG_NATIONAL_PHY is not set<br>
# CONFIG_STE10XP is not set<br>
# CONFIG_LSI_ET1011C_PHY is not set<br>
# CONFIG_MICREL_PHY is not set<br>
# CONFIG_FIXED_PHY is not set<br>
# CONFIG_MDIO_BITBANG is not set<br>
# CONFIG_MDIO_BUS_MUX_GPIO is not set<br>
# CONFIG_MDIO_BUS_MUX_MMIOREG is not set<br>
# CONFIG_MDIO_BCM_UNIMAC is not set<br>
# CONFIG_MICREL_KS8995MA is not set<br>
CONFIG_PPP=y<br>
CONFIG_PPP_BSDCOMP=y<br>
CONFIG_PPP_DEFLATE=y<br>
CONFIG_PPP_FILTER=y<br>
CONFIG_PPP_MPPE=y<br>
CONFIG_PPP_MULTILINK=y<br>
CONFIG_PPPOE=y<br>
CONFIG_PPPOL2TP=y<br>
CONFIG_PPPOLAC=y<br>
CONFIG_PPPOPNS=y<br>
CONFIG_PPP_ASYNC=y<br>
CONFIG_PPP_SYNC_TTY=y<br>
# CONFIG_SLIP is not set<br>
CONFIG_SLHC=y<br>
CONFIG_USB_NET_DRIVERS=y<br>
# CONFIG_USB_CATC is not set<br>
# CONFIG_USB_KAWETH is not set<br>
# CONFIG_USB_PEGASUS is not set<br>
# CONFIG_USB_RTL8150 is not set<br>
# CONFIG_USB_RTL8152 is not set<br>
CONFIG_USB_USBNET=y<br>
CONFIG_USB_NET_AX8817X=y<br>
CONFIG_USB_NET_AX88179_178A=y<br>
CONFIG_USB_NET_CDCETHER=y<br>
# CONFIG_USB_NET_CDC_EEM is not set<br>
CONFIG_USB_NET_CDC_NCM=y<br>
# CONFIG_USB_NET_HUAWEI_CDC_NCM is not set<br>
# CONFIG_USB_NET_CDC_MBIM is not set<br>
# CONFIG_USB_NET_DM9601 is not set<br>
# CONFIG_USB_NET_SR9700 is not set<br>
# CONFIG_USB_NET_SR9800 is not set<br>
# CONFIG_USB_NET_SMSC75XX is not set<br>
# CONFIG_USB_NET_SMSC95XX is not set<br>
# CONFIG_USB_NET_GL620A is not set<br>
CONFIG_USB_NET_NET1080=y<br>
# CONFIG_USB_NET_PLUSB is not set<br>
# CONFIG_USB_NET_MCS7830 is not set<br>
# CONFIG_USB_NET_RNDIS_HOST is not set<br>
CONFIG_USB_NET_CDC_SUBSET=y<br>
# CONFIG_USB_ALI_M5632 is not set<br>
# CONFIG_USB_AN2720 is not set<br>
CONFIG_USB_BELKIN=y<br>
CONFIG_USB_ARMLINUX=y<br>
# CONFIG_USB_EPSON2888 is not set<br>
# CONFIG_USB_KC2190 is not set<br>
CONFIG_USB_NET_ZAURUS=y<br>
# CONFIG_USB_NET_CX82310_ETH is not set<br>
# CONFIG_USB_NET_KALMIA is not set<br>
# CONFIG_USB_NET_QMI_WWAN is not set<br>
# CONFIG_USB_HSO is not set<br>
# CONFIG_USB_NET_INT51X1 is not set<br>
# CONFIG_USB_IPHETH is not set<br>
# CONFIG_USB_SIERRA_NET is not set<br>
# CONFIG_USB_VL600 is not set<br>
# CONFIG_USBNET_IPA_BRIDGE is not set<br>
CONFIG_WLAN=y<br>
# CONFIG_ATMEL is not set<br>
# CONFIG_PRISM54 is not set<br>
# CONFIG_USB_ZD1201 is not set<br>
# CONFIG_USB_NET_RNDIS_WLAN is not set<br>
# CONFIG_WIFI_CONTROL_FUNC is not set<br>
CONFIG_WCNSS_CORE=y<br>
CONFIG_WCNSS_CORE_PRONTO=y<br>
CONFIG_WCNSS_REGISTER_DUMP_ON_BITE=y<br>
CONFIG_WCNSS_MEM_PRE_ALLOC=y<br>
# CONFIG_WCNSS_SKB_PRE_ALLOC is not set<br>
CONFIG_CNSS_CRYPTO=y<br>
CONFIG_ATH_CARDS=y<br>
# CONFIG_ATH_DEBUG is not set<br>
# CONFIG_ATH5K_PCI is not set<br>
# CONFIG_ATH6KL is not set<br>
CONFIG_WIL6210=m<br>
CONFIG_WIL6210_ISR_COR=y<br>
CONFIG_WIL6210_TRACING=y<br>
# CONFIG_WIL6210_WRITE_IOCTL is not set<br>
CONFIG_WIL6210_PLATFORM_MSM=y<br>
# CONFIG_BRCMFMAC is not set<br>
# CONFIG_HOSTAP is not set<br>
# CONFIG_IPW2100 is not set<br>
# CONFIG_LIBERTAS is not set<br>
# CONFIG_WL_TI is not set<br>
# CONFIG_MWIFIEX is not set<br>
# CONFIG_CNSS is not set<br>
# CONFIG_CLD_DEBUG is not set<br>
# CONFIG_CLD_HL_SDIO_CORE is not set<br>
CONFIG_CLD_LL_CORE=y<br>
# CONFIG_CNSS_LOGGER is not set<br>
# CONFIG_WLAN_FEATURE_RX_WAKELOCK is not set<br>
<br>
#<br>
# Enable WiMAX (Networking options) to see the WiMAX drivers<br>
#<br>
# CONFIG_WAN is not set<br>
# CONFIG_VMXNET3 is not set<br>
# CONFIG_ISDN is not set<br>
<br>
#<br>
# Input device support<br>
#<br>
CONFIG_INPUT=y<br>
# CONFIG_INPUT_FF_MEMLESS is not set<br>
# CONFIG_INPUT_POLLDEV is not set<br>
# CONFIG_INPUT_SPARSEKMAP is not set<br>
# CONFIG_INPUT_MATRIXKMAP is not set<br>
<br>
#<br>
# Userland interfaces<br>
#<br>
CONFIG_INPUT_MOUSEDEV=y<br>
CONFIG_INPUT_MOUSEDEV_PSAUX=y<br>
CONFIG_INPUT_MOUSEDEV_SCREEN_X=1024<br>
CONFIG_INPUT_MOUSEDEV_SCREEN_Y=768<br>
# CONFIG_INPUT_JOYDEV is not set<br>
CONFIG_INPUT_EVDEV=y<br>
# CONFIG_INPUT_EVBUG is not set<br>
CONFIG_INPUT_KEYRESET=y<br>
CONFIG_INPUT_KEYCOMBO=y<br>
<br>
#<br>
# Input Device Drivers<br>
#<br>
CONFIG_INPUT_KEYBOARD=y<br>
# CONFIG_KEYBOARD_ADP5588 is not set<br>
# CONFIG_KEYBOARD_ADP5589 is not set<br>
CONFIG_KEYBOARD_ATKBD=y<br>
# CONFIG_KEYBOARD_QT1070 is not set<br>
# CONFIG_KEYBOARD_QT2160 is not set<br>
# CONFIG_KEYBOARD_LKKBD is not set<br>
CONFIG_KEYBOARD_GPIO=y<br>
# CONFIG_KEYBOARD_GPIO_POLLED is not set<br>
# CONFIG_KEYBOARD_TCA6416 is not set<br>
# CONFIG_KEYBOARD_TCA8418 is not set<br>
# CONFIG_KEYBOARD_MATRIX is not set<br>
# CONFIG_KEYBOARD_LM8323 is not set<br>
# CONFIG_KEYBOARD_LM8333 is not set<br>
# CONFIG_KEYBOARD_MAX7359 is not set<br>
# CONFIG_KEYBOARD_MCS is not set<br>
# CONFIG_KEYBOARD_MPR121 is not set<br>
# CONFIG_KEYBOARD_NEWTON is not set<br>
# CONFIG_KEYBOARD_OPENCORES is not set<br>
# CONFIG_KEYBOARD_SAMSUNG is not set<br>
# CONFIG_KEYBOARD_STOWAWAY is not set<br>
# CONFIG_KEYBOARD_SUNKBD is not set<br>
# CONFIG_KEYBOARD_OMAP4 is not set<br>
# CONFIG_KEYBOARD_XTKBD is not set<br>
# CONFIG_KEYBOARD_CAP1106 is not set<br>
# CONFIG_INPUT_MOUSE is not set<br>
CONFIG_INPUT_JOYSTICK=y<br>
# CONFIG_JOYSTICK_ANALOG is not set<br>
# CONFIG_JOYSTICK_A3D is not set<br>
# CONFIG_JOYSTICK_ADI is not set<br>
# CONFIG_JOYSTICK_COBRA is not set<br>
# CONFIG_JOYSTICK_GF2K is not set<br>
# CONFIG_JOYSTICK_GRIP is not set<br>
# CONFIG_JOYSTICK_GRIP_MP is not set<br>
# CONFIG_JOYSTICK_GUILLEMOT is not set<br>
# CONFIG_JOYSTICK_INTERACT is not set<br>
# CONFIG_JOYSTICK_SIDEWINDER is not set<br>
# CONFIG_JOYSTICK_TMDC is not set<br>
# CONFIG_JOYSTICK_IFORCE is not set<br>
# CONFIG_JOYSTICK_WARRIOR is not set<br>
# CONFIG_JOYSTICK_MAGELLAN is not set<br>
# CONFIG_JOYSTICK_SPACEORB is not set<br>
# CONFIG_JOYSTICK_SPACEBALL is not set<br>
# CONFIG_JOYSTICK_STINGER is not set<br>
# CONFIG_JOYSTICK_TWIDJOY is not set<br>
# CONFIG_JOYSTICK_ZHENHUA is not set<br>
# CONFIG_JOYSTICK_AS5011 is not set<br>
# CONFIG_JOYSTICK_JOYDUMP is not set<br>
CONFIG_JOYSTICK_XPAD=y<br>
# CONFIG_JOYSTICK_XPAD_FF is not set<br>
# CONFIG_JOYSTICK_XPAD_LEDS is not set<br>
CONFIG_INPUT_TABLET=y<br>
# CONFIG_TABLET_USB_ACECAD is not set<br>
# CONFIG_TABLET_USB_AIPTEK is not set<br>
# CONFIG_TABLET_USB_GTCO is not set<br>
# CONFIG_TABLET_USB_HANWANG is not set<br>
# CONFIG_TABLET_USB_KBTAB is not set<br>
# CONFIG_TABLET_SERIAL_WACOM4 is not set<br>
CONFIG_INPUT_TOUCHSCREEN=y<br>
CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_v21=y<br>
CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_I2C_v21=y<br>
# CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_SPI_v21 is not set<br>
# CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_CORE_v21 is not set<br>
CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_v26=y<br>
CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_I2C_v26=y<br>
# CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_SPI_v26 is not set<br>
# CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_RMI_HID_I2C_v26 is not set<br>
# CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_CORE_v26 is not set<br>
# CONFIG_SECURE_TOUCH_SYNAPTICS_DSX_V26 is not set<br>
CONFIG_OF_TOUCHSCREEN=y<br>
# CONFIG_TOUCHSCREEN_ADS7846 is not set<br>
# CONFIG_TOUCHSCREEN_AD7877 is not set<br>
# CONFIG_TOUCHSCREEN_AD7879 is not set<br>
# CONFIG_TOUCHSCREEN_AR1021_I2C is not set<br>
# CONFIG_TOUCHSCREEN_ATMEL_MXT is not set<br>
# CONFIG_TOUCHSCREEN_ATMEL_MAXTOUCH_TS is not set<br>
# CONFIG_TOUCHSCREEN_AUO_PIXCIR is not set<br>
# CONFIG_TOUCHSCREEN_BU21013 is not set<br>
# CONFIG_TOUCHSCREEN_CY8CTMG110 is not set<br>
# CONFIG_TOUCHSCREEN_CYTTSP_CORE is not set<br>
# CONFIG_TOUCHSCREEN_CYTTSP4_CORE is not set<br>
# CONFIG_TOUCHSCREEN_DYNAPRO is not set<br>
# CONFIG_TOUCHSCREEN_HAMPSHIRE is not set<br>
# CONFIG_TOUCHSCREEN_EETI is not set<br>
# CONFIG_TOUCHSCREEN_EGALAX is not set<br>
# CONFIG_TOUCHSCREEN_FUJITSU is not set<br>
# CONFIG_TOUCHSCREEN_ILI210X is not set<br>
# CONFIG_TOUCHSCREEN_GUNZE is not set<br>
# CONFIG_TOUCHSCREEN_ELO is not set<br>
# CONFIG_TOUCHSCREEN_WACOM_W8001 is not set<br>
# CONFIG_TOUCHSCREEN_WACOM_I2C is not set<br>
# CONFIG_TOUCHSCREEN_MAX11801 is not set<br>
# CONFIG_TOUCHSCREEN_MCS5000 is not set<br>
# CONFIG_TOUCHSCREEN_MMS114 is not set<br>
# CONFIG_TOUCHSCREEN_MTOUCH is not set<br>
# CONFIG_TOUCHSCREEN_INEXIO is not set<br>
# CONFIG_TOUCHSCREEN_MK712 is not set<br>
# CONFIG_TOUCHSCREEN_PENMOUNT is not set<br>
# CONFIG_TOUCHSCREEN_EDT_FT5X06 is not set<br>
CONFIG_TOUCHSCREEN_FT5435=y<br>
CONFIG_TOUCHSCREEN_IST3038C=y<br>
# CONFIG_TOUCHSCREEN_TOUCHRIGHT is not set<br>
# CONFIG_TOUCHSCREEN_TOUCHWIN is not set<br>
# CONFIG_TOUCHSCREEN_PIXCIR is not set<br>
# CONFIG_TOUCHSCREEN_USB_COMPOSITE is not set<br>
# CONFIG_TOUCHSCREEN_TOUCHIT213 is not set<br>
# CONFIG_TOUCHSCREEN_TSC_SERIO is not set<br>
# CONFIG_TOUCHSCREEN_TSC2005 is not set<br>
# CONFIG_TOUCHSCREEN_TSC2007 is not set<br>
# CONFIG_TOUCHSCREEN_ST1232 is not set<br>
# CONFIG_TOUCHSCREEN_SUR40 is not set<br>
# CONFIG_TOUCHSCREEN_TPS6507X is not set<br>
# CONFIG_TOUCHSCREEN_ZFORCE is not set<br>
# CONFIG_SECURE_TOUCH is not set<br>
# CONFIG_TOUCHSCREEN_IT7260_I2C is not set<br>
# CONFIG_TOUCHSCREEN_GEN_VKEYS is not set<br>
# CONFIG_TOUCHSCREEN_FT5X06 is not set<br>
# CONFIG_TOUCHSCREEN_SYNAPTICS_I2C_RMI4 is not set<br>
CONFIG_TOUCHSCREEN_GT9XX=y<br>
# CONFIG_TOUCHSCREEN_GT9XX_MIDO is not set<br>
# CONFIG_TOUCHSCREEN_MAXIM_STI is not set<br>
# CONFIG_GT9XX_TOUCHPANEL_DRIVER is not set<br>
CONFIG_TOUCHSCREEN_SWEEP2WAKE=y<br>
# CONFIG_TOUCHSCREEN_DOUBLETAP2WAKE is not set<br>
CONFIG_SWEEP2SLEEP=y<br>
CONFIG_INPUT_MISC=y<br>
# CONFIG_INPUT_AD714X is not set<br>
# CONFIG_INPUT_BMA150 is not set<br>
CONFIG_INPUT_HBTP_INPUT=y<br>
# CONFIG_HBTP_INPUT_SECURE_TOUCH is not set<br>
# CONFIG_INPUT_MMA8450 is not set<br>
# CONFIG_INPUT_MPU3050 is not set<br>
# CONFIG_INPUT_GP2A is not set<br>
# CONFIG_INPUT_GPIO_BEEPER is not set<br>
# CONFIG_INPUT_GPIO_TILT_POLLED is not set<br>
# CONFIG_INPUT_ATI_REMOTE2 is not set<br>
CONFIG_INPUT_KEYCHORD=y<br>
# CONFIG_INPUT_KEYSPAN_REMOTE is not set<br>
# CONFIG_INPUT_KXTJ9 is not set<br>
# CONFIG_INPUT_POWERMATE is not set<br>
# CONFIG_INPUT_YEALINK is not set<br>
# CONFIG_INPUT_CM109 is not set<br>
CONFIG_INPUT_UINPUT=y<br>
CONFIG_INPUT_GPIO=y<br>
# CONFIG_INPUT_PCF8574 is not set<br>
# CONFIG_INPUT_PWM_BEEPER is not set<br>
# CONFIG_INPUT_GPIO_ROTARY_ENCODER is not set<br>
# CONFIG_INPUT_ADXL34X is not set<br>
# CONFIG_INPUT_IMS_PCU is not set<br>
# CONFIG_INPUT_CMA3000 is not set<br>
# CONFIG_INPUT_SOC_BUTTON_ARRAY is not set<br>
# CONFIG_INPUT_DRV260X_HAPTICS is not set<br>
# CONFIG_INPUT_DRV2667_HAPTICS is not set<br>
CONFIG_INPUT_FINGERPRINT=y<br>
CONFIG_FINGERPRINT_GOODIX_GF3208=y<br>
CONFIG_FINGERPRINT_FPC1020=y<br>
<br>
#<br>
# Hardware I/O ports<br>
#<br>
CONFIG_SERIO=y<br>
CONFIG_SERIO_SERPORT=y<br>
# CONFIG_SERIO_AMBAKMI is not set<br>
# CONFIG_SERIO_PCIPS2 is not set<br>
CONFIG_SERIO_LIBPS2=y<br>
# CONFIG_SERIO_RAW is not set<br>
# CONFIG_SERIO_ALTERA_PS2 is not set<br>
# CONFIG_SERIO_PS2MULT is not set<br>
# CONFIG_SERIO_ARC_PS2 is not set<br>
# CONFIG_SERIO_APBPS2 is not set<br>
# CONFIG_GAMEPORT is not set<br>
<br>
#<br>
# Character devices<br>
#<br>
CONFIG_TTY=y<br>
CONFIG_VT=y<br>
CONFIG_CONSOLE_TRANSLATIONS=y<br>
CONFIG_VT_CONSOLE=y<br>
CONFIG_VT_CONSOLE_SLEEP=y<br>
CONFIG_HW_CONSOLE=y<br>
# CONFIG_VT_HW_CONSOLE_BINDING is not set<br>
CONFIG_UNIX98_PTYS=y<br>
CONFIG_DEVPTS_MULTIPLE_INSTANCES=y<br>
# CONFIG_LEGACY_PTYS is not set<br>
# CONFIG_SERIAL_NONSTANDARD is not set<br>
# CONFIG_NOZOMI is not set<br>
# CONFIG_N_GSM is not set<br>
# CONFIG_TRACE_SINK is not set<br>
# CONFIG_DEVMEM is not set<br>
# CONFIG_DEVKMEM is not set<br>
<br>
#<br>
# Serial drivers<br>
#<br>
# CONFIG_SERIAL_8250 is not set<br>
<br>
#<br>
# Non-8250 serial port support<br>
#<br>
# CONFIG_SERIAL_AMBA_PL010 is not set<br>
# CONFIG_SERIAL_AMBA_PL011 is not set<br>
# CONFIG_SERIAL_EARLYCON_ARM_SEMIHOST is not set<br>
# CONFIG_SERIAL_MAX3100 is not set<br>
# CONFIG_SERIAL_MAX310X is not set<br>
# CONFIG_SERIAL_MFD_HSU is not set<br>
CONFIG_SERIAL_CORE=y<br>
# CONFIG_SERIAL_JSM is not set<br>
CONFIG_SERIAL_MSM_HS=y<br>
# CONFIG_SERIAL_MSM_HSL is not set<br>
# CONFIG_SERIAL_SCCNXP is not set<br>
# CONFIG_SERIAL_SC16IS7XX is not set<br>
# CONFIG_SERIAL_ALTERA_JTAGUART is not set<br>
# CONFIG_SERIAL_ALTERA_UART is not set<br>
# CONFIG_SERIAL_IFX6X60 is not set<br>
CONFIG_SERIAL_MSM_SMD=y<br>
# CONFIG_SERIAL_XILINX_PS_UART is not set<br>
# CONFIG_SERIAL_ARC is not set<br>
# CONFIG_SERIAL_RP2 is not set<br>
# CONFIG_SERIAL_FSL_LPUART is not set<br>
<br>
#<br>
# Diag Support<br>
#<br>
CONFIG_DIAG_CHAR=y<br>
<br>
#<br>
# DIAG traffic over USB<br>
#<br>
CONFIG_DIAG_OVER_USB=y<br>
<br>
#<br>
# HSIC/SMUX support for DIAG<br>
#<br>
# CONFIG_TTY_PRINTK is not set<br>
# CONFIG_HVC_DCC is not set<br>
# CONFIG_IPMI_HANDLER is not set<br>
CONFIG_HW_RANDOM=y<br>
# CONFIG_HW_RANDOM_TIMERIOMEM is not set<br>
CONFIG_HW_RANDOM_MSM_LEGACY=y<br>
# CONFIG_R3964 is not set<br>
# CONFIG_APPLICOM is not set<br>
<br>
#<br>
# PCMCIA character devices<br>
#<br>
# CONFIG_RAW_DRIVER is not set<br>
# CONFIG_TCG_TPM is not set<br>
CONFIG_DEVPORT=y<br>
CONFIG_MSM_SMD_PKT=y<br>
# CONFIG_XILLYBUS is not set<br>
CONFIG_MSM_ADSPRPC=y<br>
# CONFIG_MSM_MDSP_TS is not set<br>
# CONFIG_MSM_RDBG is not set<br>
CONFIG_FRANDOM=y<br>
<br>
#<br>
# I2C support<br>
#<br>
CONFIG_I2C=y<br>
CONFIG_I2C_BOARDINFO=y<br>
CONFIG_I2C_COMPAT=y<br>
CONFIG_I2C_CHARDEV=y<br>
CONFIG_I2C_MUX=y<br>
<br>
#<br>
# Multiplexer I2C Chip support<br>
#<br>
# CONFIG_I2C_ARB_GPIO_CHALLENGE is not set<br>
# CONFIG_I2C_MUX_GPIO is not set<br>
# CONFIG_I2C_MUX_PCA9541 is not set<br>
# CONFIG_I2C_MUX_PCA954x is not set<br>
# CONFIG_I2C_MUX_PINCTRL is not set<br>
CONFIG_I2C_HELPER_AUTO=y<br>
<br>
#<br>
# I2C Hardware Bus support<br>
#<br>
<br>
#<br>
# PC SMBus host controller drivers<br>
#<br>
# CONFIG_I2C_ALI1535 is not set<br>
# CONFIG_I2C_ALI1563 is not set<br>
# CONFIG_I2C_ALI15X3 is not set<br>
# CONFIG_I2C_AMD756 is not set<br>
# CONFIG_I2C_AMD8111 is not set<br>
# CONFIG_I2C_I801 is not set<br>
# CONFIG_I2C_ISCH is not set<br>
# CONFIG_I2C_PIIX4 is not set<br>
# CONFIG_I2C_NFORCE2 is not set<br>
# CONFIG_I2C_SIS5595 is not set<br>
# CONFIG_I2C_SIS630 is not set<br>
# CONFIG_I2C_SIS96X is not set<br>
# CONFIG_I2C_VIA is not set<br>
# CONFIG_I2C_VIAPRO is not set<br>
<br>
#<br>
# I2C system bus drivers (mostly embedded / system-on-chip)<br>
#<br>
# CONFIG_I2C_CBUS_GPIO is not set<br>
# CONFIG_I2C_DESIGNWARE_PLATFORM is not set<br>
# CONFIG_I2C_DESIGNWARE_PCI is not set<br>
# CONFIG_I2C_GPIO is not set<br>
# CONFIG_I2C_NOMADIK is not set<br>
# CONFIG_I2C_OCORES is not set<br>
# CONFIG_I2C_PCA_PLATFORM is not set<br>
# CONFIG_I2C_PXA_PCI is not set<br>
# CONFIG_I2C_RK3X is not set<br>
# CONFIG_I2C_SIMTEC is not set<br>
# CONFIG_I2C_XILINX is not set<br>
# CONFIG_I2C_MSM_QUP is not set<br>
CONFIG_I2C_MSM_V2=y<br>
<br>
#<br>
# External I2C/SMBus adapter drivers<br>
#<br>
# CONFIG_I2C_DIOLAN_U2C is not set<br>
# CONFIG_I2C_PARPORT_LIGHT is not set<br>
# CONFIG_I2C_ROBOTFUZZ_OSIF is not set<br>
# CONFIG_I2C_TAOS_EVM is not set<br>
# CONFIG_I2C_TINY_USB is not set<br>
<br>
#<br>
# Other I2C/SMBus bus drivers<br>
#<br>
# CONFIG_I2C_STUB is not set<br>
# CONFIG_I2C_DEBUG_CORE is not set<br>
# CONFIG_I2C_DEBUG_ALGO is not set<br>
# CONFIG_I2C_DEBUG_BUS is not set<br>
CONFIG_SLIMBUS=y<br>
# CONFIG_SLIMBUS_MSM_CTRL is not set<br>
CONFIG_SLIMBUS_MSM_NGD=y<br>
CONFIG_SOUNDWIRE=y<br>
CONFIG_SOUNDWIRE_WCD_CTRL=y<br>
CONFIG_SPI=y<br>
# CONFIG_SPI_DEBUG is not set<br>
CONFIG_SPI_MASTER=y<br>
<br>
#<br>
# SPI Master Controller Drivers<br>
#<br>
# CONFIG_SPI_ALTERA is not set<br>
# CONFIG_SPI_BITBANG is not set<br>
# CONFIG_SPI_GPIO is not set<br>
# CONFIG_SPI_FSL_SPI is not set<br>
# CONFIG_SPI_OC_TINY is not set<br>
# CONFIG_SPI_PL022 is not set<br>
# CONFIG_SPI_PXA2XX is not set<br>
# CONFIG_SPI_PXA2XX_PCI is not set<br>
# CONFIG_SPI_ROCKCHIP is not set<br>
CONFIG_SPI_QUP=y<br>
# CONFIG_SPI_SC18IS602 is not set<br>
# CONFIG_SPI_XCOMM is not set<br>
# CONFIG_SPI_XILINX is not set<br>
# CONFIG_SPI_DESIGNWARE is not set<br>
<br>
#<br>
# SPI Protocol Masters<br>
#<br>
CONFIG_SPI_SPIDEV=y<br>
# CONFIG_SPI_TLE62X0 is not set<br>
# CONFIG_SPMI is not set<br>
# CONFIG_HSI is not set<br>
<br>
#<br>
# PPS support<br>
#<br>
# CONFIG_PPS is not set<br>
<br>
#<br>
# PPS generators support<br>
#<br>
<br>
#<br>
# PTP clock support<br>
#<br>
# CONFIG_PTP_1588_CLOCK is not set<br>
<br>
#<br>
# Enable PHYLIB and NETWORK_PHY_TIMESTAMPING to see the additional clocks.<br>
#<br>
CONFIG_PINCTRL=y<br>
<br>
#<br>
# Pin controllers<br>
#<br>
CONFIG_PINMUX=y<br>
CONFIG_PINCONF=y<br>
CONFIG_GENERIC_PINCONF=y<br>
# CONFIG_DEBUG_PINCTRL is not set<br>
# CONFIG_PINCTRL_SINGLE is not set<br>
CONFIG_PINCTRL_MSM=y<br>
# CONFIG_PINCTRL_APQ8064 is not set<br>
# CONFIG_PINCTRL_MDM9607 is not set<br>
# CONFIG_PINCTRL_MDM9640 is not set<br>
# CONFIG_PINCTRL_MDMCALIFORNIUM is not set<br>
# CONFIG_PINCTRL_APQ8084 is not set<br>
# CONFIG_PINCTRL_IPQ8064 is not set<br>
# CONFIG_PINCTRL_MSM8960 is not set<br>
# CONFIG_PINCTRL_MSM8X74 is not set<br>
# CONFIG_PINCTRL_MSM8952 is not set<br>
CONFIG_PINCTRL_MSM8953=y<br>
CONFIG_ARCH_HAVE_CUSTOM_GPIO_H=y<br>
CONFIG_ARCH_WANT_OPTIONAL_GPIOLIB=y<br>
CONFIG_ARCH_REQUIRE_GPIOLIB=y<br>
CONFIG_GPIOLIB=y<br>
CONFIG_GPIO_DEVRES=y<br>
CONFIG_OF_GPIO=y<br>
CONFIG_GPIOLIB_IRQCHIP=y<br>
# CONFIG_DEBUG_GPIO is not set<br>
CONFIG_GPIO_SYSFS=y<br>
<br>
#<br>
# Memory mapped GPIO drivers:<br>
#<br>
# CONFIG_GPIO_GENERIC_PLATFORM is not set<br>
# CONFIG_GPIO_DWAPB is not set<br>
# CONFIG_GPIO_PL061 is not set<br>
CONFIG_GPIO_QPNP_PIN=y<br>
# CONFIG_GPIO_QPNP_PIN_DEBUG is not set<br>
# CONFIG_GPIO_SCH311X is not set<br>
# CONFIG_GPIO_XGENE is not set<br>
# CONFIG_GPIO_VX855 is not set<br>
# CONFIG_GPIO_GRGPIO is not set<br>
<br>
#<br>
# I2C GPIO expanders:<br>
#<br>
# CONFIG_GPIO_MAX7300 is not set<br>
# CONFIG_GPIO_MAX732X is not set<br>
# CONFIG_GPIO_PCA953X is not set<br>
# CONFIG_GPIO_PCF857X is not set<br>
# CONFIG_GPIO_SX150X is not set<br>
# CONFIG_GPIO_ADP5588 is not set<br>
# CONFIG_GPIO_ADNP is not set<br>
<br>
#<br>
# PCI GPIO expanders:<br>
#<br>
# CONFIG_GPIO_BT8XX is not set<br>
# CONFIG_GPIO_AMD8111 is not set<br>
# CONFIG_GPIO_ML_IOH is not set<br>
# CONFIG_GPIO_RDC321X is not set<br>
<br>
#<br>
# SPI GPIO expanders:<br>
#<br>
# CONFIG_GPIO_MAX7301 is not set<br>
# CONFIG_GPIO_MCP23S08 is not set<br>
# CONFIG_GPIO_MC33880 is not set<br>
# CONFIG_GPIO_74X164 is not set<br>
<br>
#<br>
# AC97 GPIO expanders:<br>
#<br>
<br>
#<br>
# LPC GPIO expanders:<br>
#<br>
<br>
#<br>
# MODULbus GPIO expanders:<br>
#<br>
<br>
#<br>
# USB GPIO expanders:<br>
#<br>
# CONFIG_W1 is not set<br>
CONFIG_POWER_SUPPLY=y<br>
# CONFIG_POWER_SUPPLY_DEBUG is not set<br>
# CONFIG_PDA_POWER is not set<br>
# CONFIG_TEST_POWER is not set<br>
# CONFIG_BATTERY_DS2780 is not set<br>
# CONFIG_BATTERY_DS2781 is not set<br>
# CONFIG_BATTERY_DS2782 is not set<br>
# CONFIG_BATTERY_SBS is not set<br>
# CONFIG_BATTERY_BQ27x00 is not set<br>
# CONFIG_BATTERY_MAX17040 is not set<br>
# CONFIG_BATTERY_MAX17042 is not set<br>
# CONFIG_CHARGER_ISP1704 is not set<br>
# CONFIG_CHARGER_MAX8903 is not set<br>
# CONFIG_CHARGER_LP8727 is not set<br>
# CONFIG_CHARGER_GPIO is not set<br>
# CONFIG_CHARGER_MANAGER is not set<br>
# CONFIG_CHARGER_BQ2415X is not set<br>
# CONFIG_CHARGER_BQ24190 is not set<br>
# CONFIG_CHARGER_BQ24735 is not set<br>
# CONFIG_CHARGER_SMB347 is not set<br>
# CONFIG_SMB349_USB_CHARGER is not set<br>
# CONFIG_SMB349_DUAL_CHARGER is not set<br>
CONFIG_SMB1351_USB_CHARGER=y<br>
# CONFIG_SMB350_CHARGER is not set<br>
CONFIG_SMB135X_CHARGER=y<br>
# CONFIG_SMB1360_CHARGER_FG is not set<br>
# CONFIG_SMB358_CHARGER is not set<br>
# CONFIG_SMB23X_CHARGER is not set<br>
# CONFIG_BATTERY_BQ28400 is not set<br>
# CONFIG_QPNP_CHARGER is not set<br>
CONFIG_QPNP_SMBCHARGER=y<br>
# CONFIG_FUELGAUGE_STC3117 is not set<br>
CONFIG_QPNP_FG=y<br>
CONFIG_BATTERY_BCL=y<br>
# CONFIG_QPNP_VM_BMS is not set<br>
# CONFIG_QPNP_BMS is not set<br>
# CONFIG_QPNP_LINEAR_CHARGER is not set<br>
CONFIG_QPNP_TYPEC=y<br>
CONFIG_MSM_BCL_CTL=y<br>
CONFIG_MSM_BCL_PERIPHERAL_CTL=y<br>
# CONFIG_MSM_BCL_SOMC_CTL is not set<br>
CONFIG_QNS_SYSTEM=y<br>
CONFIG_POWER_RESET=y<br>
# CONFIG_POWER_RESET_GPIO is not set<br>
# CONFIG_POWER_RESET_GPIO_RESTART is not set<br>
# CONFIG_POWER_RESET_LTC2952 is not set<br>
CONFIG_POWER_RESET_MSM=y<br>
# CONFIG_MSM_DLOAD_MODE is not set<br>
CONFIG_MSM_PRESERVE_MEM=y<br>
# CONFIG_POWER_RESET_XGENE is not set<br>
# CONFIG_POWER_RESET_SYSCON is not set<br>
# CONFIG_POWER_AVS is not set<br>
CONFIG_MSM_PM=y<br>
CONFIG_APSS_CORE_EA=y<br>
CONFIG_MSM_APM=y<br>
CONFIG_MSM_IDLE_STATS=y<br>
CONFIG_MSM_IDLE_STATS_FIRST_BUCKET=62500<br>
CONFIG_MSM_IDLE_STATS_BUCKET_SHIFT=2<br>
CONFIG_MSM_IDLE_STATS_BUCKET_COUNT=10<br>
CONFIG_MSM_SUSPEND_STATS_FIRST_BUCKET=1000000000<br>
CONFIG_HWMON=y<br>
# CONFIG_HWMON_VID is not set<br>
# CONFIG_HWMON_DEBUG_CHIP is not set<br>
<br>
#<br>
# Native drivers<br>
#<br>
# CONFIG_SENSORS_AD7314 is not set<br>
# CONFIG_SENSORS_AD7414 is not set<br>
# CONFIG_SENSORS_AD7418 is not set<br>
# CONFIG_SENSORS_ADM1021 is not set<br>
# CONFIG_SENSORS_ADM1025 is not set<br>
# CONFIG_SENSORS_ADM1026 is not set<br>
# CONFIG_SENSORS_ADM1029 is not set<br>
# CONFIG_SENSORS_ADM1031 is not set<br>
# CONFIG_SENSORS_ADM9240 is not set<br>
# CONFIG_SENSORS_ADT7310 is not set<br>
# CONFIG_SENSORS_ADT7410 is not set<br>
# CONFIG_SENSORS_ADT7411 is not set<br>
# CONFIG_SENSORS_ADT7462 is not set<br>
# CONFIG_SENSORS_ADT7470 is not set<br>
# CONFIG_SENSORS_ADT7475 is not set<br>
# CONFIG_SENSORS_ASC7621 is not set<br>
# CONFIG_SENSORS_ATXP1 is not set<br>
# CONFIG_SENSORS_DS620 is not set<br>
# CONFIG_SENSORS_DS1621 is not set<br>
# CONFIG_SENSORS_I5K_AMB is not set<br>
# CONFIG_SENSORS_F71805F is not set<br>
# CONFIG_SENSORS_F71882FG is not set<br>
# CONFIG_SENSORS_F75375S is not set<br>
# CONFIG_SENSORS_GL518SM is not set<br>
# CONFIG_SENSORS_GL520SM is not set<br>
# CONFIG_SENSORS_G760A is not set<br>
# CONFIG_SENSORS_G762 is not set<br>
# CONFIG_SENSORS_GPIO_FAN is not set<br>
# CONFIG_SENSORS_HIH6130 is not set<br>
# CONFIG_SENSORS_IT87 is not set<br>
# CONFIG_SENSORS_JC42 is not set<br>
# CONFIG_SENSORS_POWR1220 is not set<br>
# CONFIG_SENSORS_LINEAGE is not set<br>
# CONFIG_SENSORS_LTC2945 is not set<br>
# CONFIG_SENSORS_LTC4151 is not set<br>
# CONFIG_SENSORS_LTC4215 is not set<br>
# CONFIG_SENSORS_LTC4222 is not set<br>
# CONFIG_SENSORS_LTC4245 is not set<br>
# CONFIG_SENSORS_LTC4260 is not set<br>
# CONFIG_SENSORS_LTC4261 is not set<br>
# CONFIG_SENSORS_MAX1111 is not set<br>
# CONFIG_SENSORS_MAX16065 is not set<br>
# CONFIG_SENSORS_MAX1619 is not set<br>
# CONFIG_SENSORS_MAX1668 is not set<br>
# CONFIG_SENSORS_MAX197 is not set<br>
# CONFIG_SENSORS_MAX6639 is not set<br>
# CONFIG_SENSORS_MAX6642 is not set<br>
# CONFIG_SENSORS_MAX6650 is not set<br>
# CONFIG_SENSORS_MAX6697 is not set<br>
# CONFIG_SENSORS_HTU21 is not set<br>
# CONFIG_SENSORS_MCP3021 is not set<br>
# CONFIG_SENSORS_ADCXX is not set<br>
# CONFIG_SENSORS_LM63 is not set<br>
# CONFIG_SENSORS_LM70 is not set<br>
# CONFIG_SENSORS_LM73 is not set<br>
# CONFIG_SENSORS_LM75 is not set<br>
# CONFIG_SENSORS_LM77 is not set<br>
# CONFIG_SENSORS_LM78 is not set<br>
# CONFIG_SENSORS_LM80 is not set<br>
# CONFIG_SENSORS_LM83 is not set<br>
# CONFIG_SENSORS_LM85 is not set<br>
# CONFIG_SENSORS_LM87 is not set<br>
# CONFIG_SENSORS_LM90 is not set<br>
# CONFIG_SENSORS_LM92 is not set<br>
# CONFIG_SENSORS_LM93 is not set<br>
# CONFIG_SENSORS_LM95234 is not set<br>
# CONFIG_SENSORS_LM95241 is not set<br>
# CONFIG_SENSORS_LM95245 is not set<br>
# CONFIG_SENSORS_PC87360 is not set<br>
# CONFIG_SENSORS_PC87427 is not set<br>
# CONFIG_SENSORS_NTC_THERMISTOR is not set<br>
# CONFIG_SENSORS_NCT6683 is not set<br>
# CONFIG_SENSORS_NCT6775 is not set<br>
# CONFIG_SENSORS_PCF8591 is not set<br>
CONFIG_SENSORS_EPM_ADC=y<br>
CONFIG_SENSORS_QPNP_ADC_VOLTAGE=y<br>
CONFIG_SENSORS_QPNP_ADC_CURRENT=y<br>
# CONFIG_PMBUS is not set<br>
# CONFIG_SENSORS_PWM_FAN is not set<br>
# CONFIG_SENSORS_SHT15 is not set<br>
# CONFIG_SENSORS_SHT21 is not set<br>
# CONFIG_SENSORS_SHTC1 is not set<br>
# CONFIG_SENSORS_SIS5595 is not set<br>
# CONFIG_SENSORS_DME1737 is not set<br>
# CONFIG_SENSORS_EMC1403 is not set<br>
# CONFIG_SENSORS_EMC2103 is not set<br>
# CONFIG_SENSORS_EMC6W201 is not set<br>
# CONFIG_SENSORS_SMSC47M1 is not set<br>
# CONFIG_SENSORS_SMSC47M192 is not set<br>
# CONFIG_SENSORS_SMSC47B397 is not set<br>
# CONFIG_SENSORS_SCH56XX_COMMON is not set<br>
# CONFIG_SENSORS_SMM665 is not set<br>
# CONFIG_SENSORS_ADC128D818 is not set<br>
# CONFIG_SENSORS_ADS1015 is not set<br>
# CONFIG_SENSORS_ADS7828 is not set<br>
# CONFIG_SENSORS_ADS7871 is not set<br>
# CONFIG_SENSORS_AMC6821 is not set<br>
# CONFIG_SENSORS_INA209 is not set<br>
# CONFIG_SENSORS_INA2XX is not set<br>
# CONFIG_SENSORS_THMC50 is not set<br>
# CONFIG_SENSORS_TMP102 is not set<br>
# CONFIG_SENSORS_TMP103 is not set<br>
# CONFIG_SENSORS_TMP401 is not set<br>
# CONFIG_SENSORS_TMP421 is not set<br>
# CONFIG_SENSORS_VIA686A is not set<br>
# CONFIG_SENSORS_VT1211 is not set<br>
# CONFIG_SENSORS_VT8231 is not set<br>
# CONFIG_SENSORS_W83781D is not set<br>
# CONFIG_SENSORS_W83791D is not set<br>
# CONFIG_SENSORS_W83792D is not set<br>
# CONFIG_SENSORS_W83793 is not set<br>
# CONFIG_SENSORS_W83795 is not set<br>
# CONFIG_SENSORS_W83L785TS is not set<br>
# CONFIG_SENSORS_W83L786NG is not set<br>
# CONFIG_SENSORS_W83627HF is not set<br>
# CONFIG_SENSORS_W83627EHF is not set<br>
CONFIG_THERMAL=y<br>
CONFIG_THERMAL_HWMON=y<br>
CONFIG_THERMAL_OF=y<br>
CONFIG_THERMAL_WRITABLE_TRIPS=y<br>
CONFIG_THERMAL_DEFAULT_GOV_STEP_WISE=y<br>
# CONFIG_THERMAL_DEFAULT_GOV_FAIR_SHARE is not set<br>
# CONFIG_THERMAL_DEFAULT_GOV_USER_SPACE is not set<br>
# CONFIG_THERMAL_DEFAULT_GOV_POWER_ALLOCATOR is not set<br>
# CONFIG_THERMAL_GOV_FAIR_SHARE is not set<br>
CONFIG_THERMAL_GOV_STEP_WISE=y<br>
# CONFIG_THERMAL_GOV_BANG_BANG is not set<br>
# CONFIG_THERMAL_GOV_USER_SPACE is not set<br>
# CONFIG_THERMAL_GOV_POWER_ALLOCATOR is not set<br>
# CONFIG_CPU_THERMAL is not set<br>
# CONFIG_THERMAL_EMULATION is not set<br>
CONFIG_THERMAL_TSENS8974=y<br>
CONFIG_LIMITS_MONITOR=y<br>
CONFIG_LIMITS_LITE_HW=y<br>
CONFIG_THERMAL_MONITOR=y<br>
CONFIG_THERMAL_QPNP=y<br>
CONFIG_THERMAL_QPNP_ADC_TM=y<br>
<br>
#<br>
# Texas Instruments thermal drivers<br>
#<br>
# CONFIG_WATCHDOG is not set<br>
CONFIG_SSB_POSSIBLE=y<br>
<br>
#<br>
# Sonics Silicon Backplane<br>
#<br>
# CONFIG_SSB is not set<br>
CONFIG_BCMA_POSSIBLE=y<br>
<br>
#<br>
# Broadcom specific AMBA<br>
#<br>
# CONFIG_BCMA is not set<br>
<br>
#<br>
# Multifunction device drivers<br>
#<br>
CONFIG_MFD_CORE=y<br>
# CONFIG_MFD_AS3711 is not set<br>
# CONFIG_MFD_AS3722 is not set<br>
# CONFIG_PMIC_ADP5520 is not set<br>
# CONFIG_MFD_AAT2870_CORE is not set<br>
# CONFIG_MFD_BCM590XX is not set<br>
# CONFIG_MFD_AXP20X is not set<br>
# CONFIG_MFD_CROS_EC is not set<br>
# CONFIG_PMIC_DA903X is not set<br>
# CONFIG_MFD_DA9052_SPI is not set<br>
# CONFIG_MFD_DA9052_I2C is not set<br>
# CONFIG_MFD_DA9055 is not set<br>
# CONFIG_MFD_DA9063 is not set<br>
# CONFIG_MFD_MC13XXX_SPI is not set<br>
# CONFIG_MFD_MC13XXX_I2C is not set<br>
# CONFIG_MFD_HI6421_PMIC is not set<br>
# CONFIG_HTC_PASIC3 is not set<br>
# CONFIG_HTC_I2CPLD is not set<br>
# CONFIG_LPC_ICH is not set<br>
# CONFIG_LPC_SCH is not set<br>
# CONFIG_INTEL_SOC_PMIC is not set<br>
# CONFIG_MFD_JANZ_CMODIO is not set<br>
# CONFIG_MFD_KEMPLD is not set<br>
# CONFIG_MFD_88PM800 is not set<br>
# CONFIG_MFD_88PM805 is not set<br>
# CONFIG_MFD_88PM860X is not set<br>
# CONFIG_MFD_MAX14577 is not set<br>
# CONFIG_MFD_MAX77686 is not set<br>
# CONFIG_MFD_MAX77693 is not set<br>
# CONFIG_MFD_MAX8907 is not set<br>
# CONFIG_MFD_MAX8925 is not set<br>
# CONFIG_MFD_MAX8997 is not set<br>
# CONFIG_MFD_MAX8998 is not set<br>
# CONFIG_MFD_MENF21BMC is not set<br>
# CONFIG_EZX_PCAP is not set<br>
# CONFIG_MFD_VIPERBOARD is not set<br>
# CONFIG_MFD_RETU is not set<br>
# CONFIG_MFD_PCF50633 is not set<br>
# CONFIG_MFD_I2C_PMIC is not set<br>
# CONFIG_MFD_RDC321X is not set<br>
# CONFIG_MFD_RTSX_PCI is not set<br>
# CONFIG_MFD_RTSX_USB is not set<br>
# CONFIG_MFD_RC5T583 is not set<br>
# CONFIG_MFD_RK808 is not set<br>
# CONFIG_MFD_RN5T618 is not set<br>
# CONFIG_MFD_SEC_CORE is not set<br>
# CONFIG_MFD_SI476X_CORE is not set<br>
# CONFIG_MFD_SM501 is not set<br>
# CONFIG_MFD_SMSC is not set<br>
# CONFIG_ABX500_CORE is not set<br>
# CONFIG_MFD_STMPE is not set<br>
# CONFIG_MFD_SYSCON is not set<br>
# CONFIG_MFD_TI_AM335X_TSCADC is not set<br>
# CONFIG_MFD_LP3943 is not set<br>
# CONFIG_MFD_LP8788 is not set<br>
# CONFIG_MFD_PALMAS is not set<br>
# CONFIG_TPS6105X is not set<br>
# CONFIG_TPS65010 is not set<br>
# CONFIG_TPS6507X is not set<br>
# CONFIG_MFD_TPS65090 is not set<br>
# CONFIG_MFD_TPS65217 is not set<br>
# CONFIG_MFD_TPS65218 is not set<br>
# CONFIG_MFD_TPS6586X is not set<br>
# CONFIG_MFD_TPS65910 is not set<br>
# CONFIG_MFD_TPS65912 is not set<br>
# CONFIG_MFD_TPS65912_I2C is not set<br>
# CONFIG_MFD_TPS65912_SPI is not set<br>
# CONFIG_MFD_TPS80031 is not set<br>
# CONFIG_TWL4030_CORE is not set<br>
# CONFIG_TWL6040_CORE is not set<br>
# CONFIG_MFD_WL1273_CORE is not set<br>
# CONFIG_MFD_LM3533 is not set<br>
# CONFIG_MFD_TC3589X is not set<br>
# CONFIG_MFD_TMIO is not set<br>
# CONFIG_MFD_VX855 is not set<br>
# CONFIG_MFD_ARIZONA_I2C is not set<br>
# CONFIG_MFD_ARIZONA_SPI is not set<br>
# CONFIG_MFD_WM8400 is not set<br>
# CONFIG_MFD_WM831X_I2C is not set<br>
# CONFIG_MFD_WM831X_SPI is not set<br>
# CONFIG_MFD_WM8350_I2C is not set<br>
# CONFIG_MFD_WM8994 is not set<br>
# CONFIG_WCD9306_CODEC is not set<br>
# CONFIG_WCD9320_CODEC is not set<br>
# CONFIG_WCD9330_CODEC is not set<br>
CONFIG_WCD9335_CODEC=y<br>
CONFIG_REGULATOR=y<br>
# CONFIG_REGULATOR_DEBUG is not set<br>
CONFIG_REGULATOR_FIXED_VOLTAGE=y<br>
# CONFIG_REGULATOR_VIRTUAL_CONSUMER is not set<br>
# CONFIG_REGULATOR_USERSPACE_CONSUMER is not set<br>
# CONFIG_REGULATOR_PROXY_CONSUMER is not set<br>
# CONFIG_REGULATOR_ACT8865 is not set<br>
# CONFIG_REGULATOR_AD5398 is not set<br>
CONFIG_REGULATOR_STUB=y<br>
# CONFIG_REGULATOR_DA9210 is not set<br>
# CONFIG_REGULATOR_DA9211 is not set<br>
CONFIG_REGULATOR_FAN53555=y<br>
CONFIG_REGULATOR_MSM_GFX_LDO=y<br>
# CONFIG_REGULATOR_GPIO is not set<br>
# CONFIG_REGULATOR_ISL9305 is not set<br>
CONFIG_REGULATOR_MEM_ACC=y<br>
# CONFIG_REGULATOR_ISL6271A is not set<br>
# CONFIG_REGULATOR_LP3971 is not set<br>
# CONFIG_REGULATOR_LP3972 is not set<br>
# CONFIG_REGULATOR_LP872X is not set<br>
# CONFIG_REGULATOR_LP8755 is not set<br>
# CONFIG_REGULATOR_LTC3589 is not set<br>
# CONFIG_REGULATOR_MAX1586 is not set<br>
# CONFIG_REGULATOR_MAX8649 is not set<br>
# CONFIG_REGULATOR_MAX8660 is not set<br>
# CONFIG_REGULATOR_MAX8952 is not set<br>
# CONFIG_REGULATOR_MAX8973 is not set<br>
# CONFIG_REGULATOR_PFUZE100 is not set<br>
# CONFIG_REGULATOR_PWM is not set<br>
# CONFIG_REGULATOR_TPS51632 is not set<br>
# CONFIG_REGULATOR_TPS62360 is not set<br>
# CONFIG_REGULATOR_TPS65023 is not set<br>
# CONFIG_REGULATOR_TPS6507X is not set<br>
# CONFIG_REGULATOR_TPS6524X is not set<br>
CONFIG_REGULATOR_RPM_SMD=y<br>
CONFIG_REGULATOR_QPNP=y<br>
CONFIG_REGULATOR_QPNP_LABIBB=y<br>
CONFIG_REGULATOR_SPM=y<br>
CONFIG_REGULATOR_CPR=y<br>
# CONFIG_REGULATOR_CPR2_GFX is not set<br>
CONFIG_REGULATOR_CPR3=y<br>
CONFIG_REGULATOR_CPR3_HMSS=y<br>
CONFIG_REGULATOR_CPR3_MMSS=y<br>
CONFIG_REGULATOR_CPR4_APSS=y<br>
CONFIG_REGULATOR_CPRH_KBSS=y<br>
CONFIG_REGULATOR_KRYO=y<br>
CONFIG_MEDIA_SUPPORT=y<br>
<br>
#<br>
# Multimedia core support<br>
#<br>
CONFIG_MEDIA_CAMERA_SUPPORT=y<br>
# CONFIG_MEDIA_ANALOG_TV_SUPPORT is not set<br>
# CONFIG_MEDIA_DIGITAL_TV_SUPPORT is not set<br>
CONFIG_MEDIA_RADIO_SUPPORT=y<br>
# CONFIG_MEDIA_SDR_SUPPORT is not set<br>
CONFIG_MEDIA_RC_SUPPORT=y<br>
CONFIG_MEDIA_CONTROLLER=y<br>
CONFIG_VIDEO_DEV=y<br>
CONFIG_VIDEO_V4L2_SUBDEV_API=y<br>
CONFIG_VIDEO_V4L2=y<br>
# CONFIG_VIDEO_ADV_DEBUG is not set<br>
# CONFIG_VIDEO_FIXED_MINOR_RANGES is not set<br>
CONFIG_V4L2_MEM2MEM_DEV=y<br>
CONFIG_VIDEOBUF_GEN=y<br>
CONFIG_VIDEOBUF2_CORE=y<br>
CONFIG_VIDEOBUF2_MEMOPS=y<br>
CONFIG_VIDEOBUF2_VMALLOC=y<br>
# CONFIG_TTPCI_EEPROM is not set<br>
<br>
#<br>
# Media drivers<br>
#<br>
CONFIG_RC_CORE=y<br>
CONFIG_RC_MAP=y<br>
CONFIG_RC_DECODERS=y<br>
CONFIG_LIRC=y<br>
CONFIG_IR_LIRC_CODEC=y<br>
CONFIG_IR_NEC_DECODER=y<br>
CONFIG_IR_RC5_DECODER=y<br>
CONFIG_IR_RC6_DECODER=y<br>
CONFIG_IR_JVC_DECODER=y<br>
CONFIG_IR_SONY_DECODER=y<br>
CONFIG_IR_SANYO_DECODER=y<br>
CONFIG_IR_SHARP_DECODER=y<br>
# CONFIG_IR_MCE_KBD_DECODER is not set<br>
CONFIG_IR_XMP_DECODER=y<br>
# CONFIG_IR_DUMP_DECODER is not set<br>
CONFIG_RC_DEVICES=y<br>
# CONFIG_RC_ATI_REMOTE is not set<br>
# CONFIG_IR_HIX5HD2 is not set<br>
# CONFIG_IR_IMON is not set<br>
# CONFIG_IR_MCEUSB is not set<br>
# CONFIG_IR_REDRAT3 is not set<br>
# CONFIG_IR_STREAMZAP is not set<br>
# CONFIG_IR_IGUANA is not set<br>
# CONFIG_IR_TTUSBIR is not set<br>
# CONFIG_IR_IMG is not set<br>
# CONFIG_RC_LOOPBACK is not set<br>
# CONFIG_IR_GPIO_CIR is not set<br>
# CONFIG_IR_GPIO is not set<br>
CONFIG_IR_PWM=y<br>
# CONFIG_MEDIA_USB_SUPPORT is not set<br>
# CONFIG_MEDIA_PCI_SUPPORT is not set<br>
CONFIG_V4L_PLATFORM_DRIVERS=y<br>
# CONFIG_VIDEO_CAFE_CCIC is not set<br>
CONFIG_SOC_CAMERA=y<br>
CONFIG_SOC_CAMERA_PLATFORM=y<br>
# CONFIG_V4L_MEM2MEM_DRIVERS is not set<br>
# CONFIG_V4L_TEST_DRIVERS is not set<br>
CONFIG_MSM_VIDC_V4L2=y<br>
CONFIG_MSM_VIDC_VMEM=y<br>
CONFIG_MSM_VIDC_GOVERNORS=y<br>
<br>
#<br>
# Qualcomm MSM Camera And Video<br>
#<br>
CONFIG_MSM_CAMERA=y<br>
# CONFIG_MSM_CAMERA_DEBUG is not set<br>
# CONFIG_MSM_CAMERA_AUTOMOTIVE is not set<br>
CONFIG_MSMB_CAMERA=y<br>
# CONFIG_MSMB_CAMERA_DEBUG is not set<br>
CONFIG_MSM_CAMERA_SENSOR=y<br>
CONFIG_MSM_CPP=y<br>
CONFIG_MSM_CCI=y<br>
CONFIG_MSM_CSI20_HEADER=y<br>
CONFIG_MSM_CSI22_HEADER=y<br>
CONFIG_MSM_CSI30_HEADER=y<br>
CONFIG_MSM_CSI31_HEADER=y<br>
CONFIG_MSM_CSIPHY=y<br>
CONFIG_MSM_CSID=y<br>
CONFIG_MSM_EEPROM=y<br>
CONFIG_MSM_ISPIF=y<br>
# CONFIG_MSM_ISPIF_V1 is not set<br>
# CONFIG_MSM_ISPIF_V2 is not set<br>
CONFIG_IMX134=y<br>
CONFIG_IMX132=y<br>
CONFIG_OV9724=y<br>
CONFIG_OV5648=y<br>
CONFIG_GC0339=y<br>
CONFIG_OV8825=y<br>
CONFIG_OV8865=y<br>
CONFIG_s5k4e1=y<br>
CONFIG_OV12830=y<br>
CONFIG_MSM_V4L2_VIDEO_OVERLAY_DEVICE=y<br>
CONFIG_MSMB_JPEG=y<br>
CONFIG_MSM_FD=y<br>
# CONFIG_MSM_JPEGDMA is not set<br>
# CONFIG_TSPP is not set<br>
CONFIG_MSM_SDE_ROTATOR=y<br>
<br>
#<br>
# Supported MMC/SDIO adapters<br>
#<br>
CONFIG_RADIO_ADAPTERS=y<br>
# CONFIG_RADIO_SI470X is not set<br>
# CONFIG_RADIO_SI4713 is not set<br>
# CONFIG_USB_MR800 is not set<br>
# CONFIG_USB_DSBR is not set<br>
# CONFIG_RADIO_MAXIRADIO is not set<br>
# CONFIG_RADIO_SHARK is not set<br>
# CONFIG_RADIO_SHARK2 is not set<br>
# CONFIG_USB_KEENE is not set<br>
# CONFIG_USB_RAREMONO is not set<br>
# CONFIG_USB_MA901 is not set<br>
# CONFIG_RADIO_TEA5764 is not set<br>
# CONFIG_RADIO_SAA7706H is not set<br>
# CONFIG_RADIO_TEF6862 is not set<br>
# CONFIG_RADIO_WL1273 is not set<br>
<br>
#<br>
# Texas Instruments WL128x FM driver (ST based)<br>
#<br>
# CONFIG_RADIO_WL128X is not set<br>
CONFIG_RADIO_IRIS=y<br>
CONFIG_RADIO_IRIS_TRANSPORT=y<br>
CONFIG_RADIO_SILABS=y<br>
# CONFIG_CYPRESS_FIRMWARE is not set<br>
<br>
#<br>
# Media ancillary drivers (tuners, sensors, i2c, frontends)<br>
#<br>
CONFIG_MEDIA_SUBDRV_AUTOSELECT=y<br>
CONFIG_MEDIA_ATTACH=y<br>
CONFIG_VIDEO_IR_I2C=y<br>
<br>
#<br>
# Audio decoders, processors and mixers<br>
#<br>
<br>
#<br>
# RDS decoders<br>
#<br>
<br>
#<br>
# Video decoders<br>
#<br>
<br>
#<br>
# Video and audio decoders<br>
#<br>
<br>
#<br>
# Video encoders<br>
#<br>
<br>
#<br>
# Camera sensor devices<br>
#<br>
<br>
#<br>
# Flash devices<br>
#<br>
<br>
#<br>
# Video improvement chips<br>
#<br>
<br>
#<br>
# Audio/Video compression chips<br>
#<br>
<br>
#<br>
# Miscellaneous helper chips<br>
#<br>
<br>
#<br>
# Sensors used on soc_camera driver<br>
#<br>
<br>
#<br>
# soc_camera sensor drivers<br>
#<br>
# CONFIG_SOC_CAMERA_IMX074 is not set<br>
# CONFIG_SOC_CAMERA_MT9M001 is not set<br>
# CONFIG_SOC_CAMERA_MT9M111 is not set<br>
# CONFIG_SOC_CAMERA_MT9T031 is not set<br>
# CONFIG_SOC_CAMERA_MT9T112 is not set<br>
# CONFIG_SOC_CAMERA_MT9V022 is not set<br>
# CONFIG_SOC_CAMERA_OV2640 is not set<br>
# CONFIG_SOC_CAMERA_OV5642 is not set<br>
# CONFIG_SOC_CAMERA_OV6650 is not set<br>
# CONFIG_SOC_CAMERA_OV772X is not set<br>
# CONFIG_SOC_CAMERA_OV9640 is not set<br>
# CONFIG_SOC_CAMERA_OV9740 is not set<br>
# CONFIG_SOC_CAMERA_RJ54N1 is not set<br>
# CONFIG_SOC_CAMERA_TW9910 is not set<br>
CONFIG_MEDIA_TUNER=y<br>
CONFIG_MEDIA_TUNER_SIMPLE=y<br>
CONFIG_MEDIA_TUNER_TDA8290=y<br>
CONFIG_MEDIA_TUNER_TDA827X=y<br>
CONFIG_MEDIA_TUNER_TDA18271=y<br>
CONFIG_MEDIA_TUNER_TDA9887=y<br>
CONFIG_MEDIA_TUNER_TEA5761=y<br>
CONFIG_MEDIA_TUNER_TEA5767=y<br>
CONFIG_MEDIA_TUNER_MT20XX=y<br>
CONFIG_MEDIA_TUNER_XC2028=y<br>
CONFIG_MEDIA_TUNER_XC5000=y<br>
CONFIG_MEDIA_TUNER_XC4000=y<br>
CONFIG_MEDIA_TUNER_MC44S803=y<br>
<br>
#<br>
# Tools to develop new frontends<br>
#<br>
# CONFIG_DVB_DUMMY_FE is not set<br>
<br>
#<br>
# Graphics support<br>
#<br>
CONFIG_VGA_ARB=y<br>
CONFIG_VGA_ARB_MAX_GPUS=16<br>
CONFIG_MSM_KGSL=y<br>
# CONFIG_MSM_KGSL_CFF_DUMP is not set<br>
CONFIG_MSM_ADRENO_DEFAULT_GOVERNOR="msm-adreno-tz"<br>
CONFIG_MSM_KGSL_IOMMU=y<br>
<br>
#<br>
# Direct Rendering Manager<br>
#<br>
# CONFIG_DRM is not set<br>
<br>
#<br>
# Frame buffer Devices<br>
#<br>
CONFIG_FB=y<br>
# CONFIG_FIRMWARE_EDID is not set<br>
CONFIG_FB_CMDLINE=y<br>
# CONFIG_FB_DDC is not set<br>
# CONFIG_FB_BOOT_VESA_SUPPORT is not set<br>
CONFIG_FB_CFB_FILLRECT=y<br>
CONFIG_FB_CFB_COPYAREA=y<br>
CONFIG_FB_CFB_IMAGEBLIT=y<br>
# CONFIG_FB_CFB_REV_PIXELS_IN_BYTE is not set<br>
# CONFIG_FB_SYS_FILLRECT is not set<br>
# CONFIG_FB_SYS_COPYAREA is not set<br>
# CONFIG_FB_SYS_IMAGEBLIT is not set<br>
# CONFIG_FB_FOREIGN_ENDIAN is not set<br>
# CONFIG_FB_SYS_FOPS is not set<br>
# CONFIG_FB_SVGALIB is not set<br>
# CONFIG_FB_MACMODES is not set<br>
# CONFIG_FB_BACKLIGHT is not set<br>
# CONFIG_FB_MODE_HELPERS is not set<br>
# CONFIG_FB_TILEBLITTING is not set<br>
<br>
#<br>
# Frame buffer hardware drivers<br>
#<br>
# CONFIG_FB_CIRRUS is not set<br>
# CONFIG_FB_PM2 is not set<br>
# CONFIG_FB_ARMCLCD is not set<br>
# CONFIG_FB_CYBER2000 is not set<br>
# CONFIG_FB_ASILIANT is not set<br>
# CONFIG_FB_IMSTT is not set<br>
# CONFIG_FB_OPENCORES is not set<br>
# CONFIG_FB_S1D13XXX is not set<br>
# CONFIG_FB_NVIDIA is not set<br>
# CONFIG_FB_RIVA is not set<br>
# CONFIG_FB_I740 is not set<br>
# CONFIG_FB_MATROX is not set<br>
# CONFIG_FB_RADEON is not set<br>
# CONFIG_FB_ATY128 is not set<br>
# CONFIG_FB_ATY is not set<br>
# CONFIG_FB_S3 is not set<br>
# CONFIG_FB_SAVAGE is not set<br>
# CONFIG_FB_SIS is not set<br>
# CONFIG_FB_NEOMAGIC is not set<br>
# CONFIG_FB_KYRO is not set<br>
# CONFIG_FB_3DFX is not set<br>
# CONFIG_FB_VOODOO1 is not set<br>
# CONFIG_FB_VT8623 is not set<br>
# CONFIG_FB_TRIDENT is not set<br>
# CONFIG_FB_ARK is not set<br>
# CONFIG_FB_PM3 is not set<br>
# CONFIG_FB_CARMINE is not set<br>
# CONFIG_FB_SMSCUFX is not set<br>
# CONFIG_FB_UDL is not set<br>
# CONFIG_FB_VIRTUAL is not set<br>
# CONFIG_FB_METRONOME is not set<br>
# CONFIG_FB_MB862XX is not set<br>
CONFIG_FB_MSM=y<br>
# CONFIG_FB_BROADSHEET is not set<br>
# CONFIG_FB_AUO_K190X is not set<br>
# CONFIG_FB_SIMPLE is not set<br>
# CONFIG_MSM_BA_V4L2 is not set<br>
CONFIG_MSM_DBA=y<br>
CONFIG_MSM_DBA_ADV7533=y<br>
CONFIG_FB_MSM_MDSS_COMMON=y<br>
# CONFIG_FB_MSM_MDP is not set<br>
CONFIG_FB_MSM_MDSS=y<br>
# CONFIG_FB_MSM_MDP_NONE is not set<br>
# CONFIG_FB_MSM_QPIC_ILI_QVGA_PANEL is not set<br>
# CONFIG_FB_MSM_QPIC_PANEL_DETECT is not set<br>
CONFIG_FB_MSM_MDSS_WRITEBACK=y<br>
CONFIG_FB_MSM_MDSS_HDMI_PANEL=y<br>
# CONFIG_FB_MSM_MDSS_HDMI_MHL_SII8334 is not set<br>
# CONFIG_FB_MSM_MDSS_MHL3 is not set<br>
# CONFIG_FB_MSM_MDSS_DSI_CTRL_STATUS is not set<br>
# CONFIG_FB_MSM_MDSS_EDP_PANEL is not set<br>
# CONFIG_FB_MSM_MDSS_MDP3 is not set<br>
CONFIG_FB_MSM_MDSS_XLOG_DEBUG=y<br>
CONFIG_FB_MSM_MDSS_KCAL_CTRL=y<br>
# CONFIG_FB_SSD1307 is not set<br>
# CONFIG_BACKLIGHT_LCD_SUPPORT is not set<br>
# CONFIG_ADF is not set<br>
# CONFIG_VGASTATE is not set<br>
<br>
#<br>
# Console display driver support<br>
#<br>
CONFIG_DUMMY_CONSOLE=y<br>
# CONFIG_FRAMEBUFFER_CONSOLE is not set<br>
# CONFIG_LOGO is not set<br>
CONFIG_SOUND=y<br>
# CONFIG_SOUND_OSS_CORE is not set<br>
CONFIG_SND=y<br>
CONFIG_SND_TIMER=y<br>
CONFIG_SND_PCM=y<br>
CONFIG_SND_HWDEP=y<br>
CONFIG_SND_RAWMIDI=y<br>
CONFIG_SND_COMPRESS_OFFLOAD=y<br>
CONFIG_SND_JACK=y<br>
# CONFIG_SND_SEQUENCER is not set<br>
# CONFIG_SND_MIXER_OSS is not set<br>
# CONFIG_SND_PCM_OSS is not set<br>
# CONFIG_SND_HRTIMER is not set<br>
CONFIG_SND_DYNAMIC_MINORS=y<br>
CONFIG_SND_MAX_CARDS=32<br>
CONFIG_SND_SUPPORT_OLD_API=y<br>
CONFIG_SND_VERBOSE_PROCFS=y<br>
# CONFIG_SND_VERBOSE_PRINTK is not set<br>
# CONFIG_SND_DEBUG is not set<br>
# CONFIG_SND_RAWMIDI_SEQ is not set<br>
# CONFIG_SND_OPL3_LIB_SEQ is not set<br>
# CONFIG_SND_OPL4_LIB_SEQ is not set<br>
# CONFIG_SND_SBAWE_SEQ is not set<br>
# CONFIG_SND_EMU10K1_SEQ is not set<br>
CONFIG_SND_DRIVERS=y<br>
# CONFIG_SND_DUMMY is not set<br>
# CONFIG_SND_ALOOP is not set<br>
# CONFIG_SND_MTPAV is not set<br>
# CONFIG_SND_SERIAL_U16550 is not set<br>
# CONFIG_SND_MPU401 is not set<br>
CONFIG_SND_PCI=y<br>
# CONFIG_SND_AD1889 is not set<br>
# CONFIG_SND_ALS300 is not set<br>
# CONFIG_SND_ALI5451 is not set<br>
# CONFIG_SND_ATIIXP is not set<br>
# CONFIG_SND_ATIIXP_MODEM is not set<br>
# CONFIG_SND_AU8810 is not set<br>
# CONFIG_SND_AU8820 is not set<br>
# CONFIG_SND_AU8830 is not set<br>
# CONFIG_SND_AW2 is not set<br>
# CONFIG_SND_AZT3328 is not set<br>
# CONFIG_SND_BT87X is not set<br>
# CONFIG_SND_CA0106 is not set<br>
# CONFIG_SND_CMIPCI is not set<br>
# CONFIG_SND_OXYGEN is not set<br>
# CONFIG_SND_CS4281 is not set<br>
# CONFIG_SND_CS46XX is not set<br>
# CONFIG_SND_CTXFI is not set<br>
# CONFIG_SND_DARLA20 is not set<br>
# CONFIG_SND_GINA20 is not set<br>
# CONFIG_SND_LAYLA20 is not set<br>
# CONFIG_SND_DARLA24 is not set<br>
# CONFIG_SND_GINA24 is not set<br>
# CONFIG_SND_LAYLA24 is not set<br>
# CONFIG_SND_MONA is not set<br>
# CONFIG_SND_MIA is not set<br>
# CONFIG_SND_ECHO3G is not set<br>
# CONFIG_SND_INDIGO is not set<br>
# CONFIG_SND_INDIGOIO is not set<br>
# CONFIG_SND_INDIGODJ is not set<br>
# CONFIG_SND_INDIGOIOX is not set<br>
# CONFIG_SND_INDIGODJX is not set<br>
# CONFIG_SND_EMU10K1 is not set<br>
# CONFIG_SND_EMU10K1X is not set<br>
# CONFIG_SND_ENS1370 is not set<br>
# CONFIG_SND_ENS1371 is not set<br>
# CONFIG_SND_ES1938 is not set<br>
# CONFIG_SND_ES1968 is not set<br>
# CONFIG_SND_FM801 is not set<br>
# CONFIG_SND_HDSP is not set<br>
# CONFIG_SND_HDSPM is not set<br>
# CONFIG_SND_ICE1712 is not set<br>
# CONFIG_SND_ICE1724 is not set<br>
# CONFIG_SND_INTEL8X0 is not set<br>
# CONFIG_SND_INTEL8X0M is not set<br>
# CONFIG_SND_KORG1212 is not set<br>
# CONFIG_SND_LOLA is not set<br>
# CONFIG_SND_LX6464ES is not set<br>
# CONFIG_SND_MAESTRO3 is not set<br>
# CONFIG_SND_MIXART is not set<br>
# CONFIG_SND_NM256 is not set<br>
# CONFIG_SND_PCXHR is not set<br>
# CONFIG_SND_RIPTIDE is not set<br>
# CONFIG_SND_RME32 is not set<br>
# CONFIG_SND_RME96 is not set<br>
# CONFIG_SND_RME9652 is not set<br>
# CONFIG_SND_SONICVIBES is not set<br>
# CONFIG_SND_TRIDENT is not set<br>
# CONFIG_SND_VIA82XX is not set<br>
# CONFIG_SND_VIA82XX_MODEM is not set<br>
# CONFIG_SND_VIRTUOSO is not set<br>
# CONFIG_SND_VX222 is not set<br>
# CONFIG_SND_YMFPCI is not set<br>
<br>
#<br>
# HD-Audio<br>
#<br>
# CONFIG_SND_HDA_INTEL is not set<br>
CONFIG_SND_SPI=y<br>
CONFIG_SND_USB=y<br>
CONFIG_SND_USB_AUDIO=y<br>
# CONFIG_SND_USB_UA101 is not set<br>
# CONFIG_SND_USB_CAIAQ is not set<br>
# CONFIG_SND_USB_6FIRE is not set<br>
# CONFIG_SND_USB_HIFACE is not set<br>
# CONFIG_SND_BCD2000 is not set<br>
# CONFIG_SND_USB_AUDIO_QMI is not set<br>
CONFIG_SND_SOC=y<br>
# CONFIG_SND_ATMEL_SOC is not set<br>
# CONFIG_SND_DESIGNWARE_I2S is not set<br>
<br>
#<br>
# SoC Audio for Freescale CPUs<br>
#<br>
<br>
#<br>
# Common SoC Audio options for Freescale CPUs:<br>
#<br>
# CONFIG_SND_SOC_FSL_ASRC is not set<br>
# CONFIG_SND_SOC_FSL_SAI is not set<br>
# CONFIG_SND_SOC_FSL_SSI is not set<br>
# CONFIG_SND_SOC_FSL_SPDIF is not set<br>
# CONFIG_SND_SOC_FSL_ESAI is not set<br>
# CONFIG_SND_SOC_IMX_AUDMUX is not set<br>
<br>
#<br>
# MSM SoC Audio support<br>
#<br>
CONFIG_SND_SOC_MSM_HOSTLESS_PCM=y<br>
CONFIG_SND_SOC_MSM_QDSP6V2_INTF=y<br>
CONFIG_SND_SOC_QDSP6V2=y<br>
# CONFIG_SND_SOC_QDSP_DEBUG is not set<br>
CONFIG_DOLBY_DAP=y<br>
CONFIG_DOLBY_DS2=y<br>
CONFIG_DTS_EAGLE=y<br>
CONFIG_DTS_SRS_TM=y<br>
CONFIG_QTI_PP=y<br>
CONFIG_SND_SOC_CPE=y<br>
CONFIG_SND_SOC_MSM8X16=y<br>
CONFIG_SND_SOC_I2C_AND_SPI=y<br>
CONFIG_SOUND_CONTROL=y<br>
<br>
#<br>
# CODEC drivers<br>
#<br>
# CONFIG_SND_SOC_ADAU1701 is not set<br>
# CONFIG_SND_SOC_AK4104 is not set<br>
# CONFIG_SND_SOC_AK4554 is not set<br>
# CONFIG_SND_SOC_AK4642 is not set<br>
# CONFIG_SND_SOC_AK5386 is not set<br>
# CONFIG_SND_SOC_ALC5623 is not set<br>
# CONFIG_SND_SOC_CS35L32 is not set<br>
# CONFIG_SND_SOC_CS42L52 is not set<br>
# CONFIG_SND_SOC_CS42L56 is not set<br>
# CONFIG_SND_SOC_CS42L73 is not set<br>
# CONFIG_SND_SOC_CS4265 is not set<br>
# CONFIG_SND_SOC_CS4270 is not set<br>
# CONFIG_SND_SOC_CS4271 is not set<br>
# CONFIG_SND_SOC_CS42XX8_I2C is not set<br>
# CONFIG_SND_SOC_HDMI_CODEC is not set<br>
# CONFIG_SND_SOC_ES8328 is not set<br>
# CONFIG_SND_SOC_PCM1681 is not set<br>
# CONFIG_SND_SOC_PCM1792A is not set<br>
# CONFIG_SND_SOC_PCM512x_I2C is not set<br>
# CONFIG_SND_SOC_PCM512x_SPI is not set<br>
# CONFIG_SND_SOC_SGTL5000 is not set<br>
# CONFIG_SND_SOC_SIRF_AUDIO_CODEC is not set<br>
# CONFIG_SND_SOC_SPDIF is not set<br>
# CONFIG_SND_SOC_SSM2602_SPI is not set<br>
# CONFIG_SND_SOC_SSM2602_I2C is not set<br>
# CONFIG_SND_SOC_SSM4567 is not set<br>
# CONFIG_SND_SOC_STA350 is not set<br>
# CONFIG_SND_SOC_TAS2552 is not set<br>
# CONFIG_SND_SOC_TAS5086 is not set<br>
# CONFIG_SND_SOC_TLV320AIC31XX is not set<br>
# CONFIG_SND_SOC_TLV320AIC3X is not set<br>
CONFIG_SND_SOC_WCD9330=y<br>
CONFIG_SND_SOC_WCD9335=y<br>
CONFIG_SND_SOC_WSA881X_SENSORS=y<br>
CONFIG_SND_SOC_WSA881X=y<br>
CONFIG_SND_SOC_WSA881X_ANALOG=y<br>
CONFIG_SND_SOC_MSM8X16_WCD=y<br>
CONFIG_SND_SOC_WCD9XXX=y<br>
CONFIG_SND_SOC_WCD9XXX_V2=y<br>
CONFIG_SND_SOC_WCD_CPE=y<br>
CONFIG_AUDIO_EXT_CLK=y<br>
CONFIG_SND_SOC_WCD_MBHC=y<br>
# CONFIG_SND_SOC_WM8510 is not set<br>
# CONFIG_SND_SOC_WM8523 is not set<br>
# CONFIG_SND_SOC_WM8580 is not set<br>
# CONFIG_SND_SOC_WM8711 is not set<br>
# CONFIG_SND_SOC_WM8728 is not set<br>
# CONFIG_SND_SOC_WM8731 is not set<br>
# CONFIG_SND_SOC_WM8737 is not set<br>
# CONFIG_SND_SOC_WM8741 is not set<br>
# CONFIG_SND_SOC_WM8750 is not set<br>
# CONFIG_SND_SOC_WM8753 is not set<br>
# CONFIG_SND_SOC_WM8770 is not set<br>
# CONFIG_SND_SOC_WM8776 is not set<br>
# CONFIG_SND_SOC_WM8804 is not set<br>
# CONFIG_SND_SOC_WM8903 is not set<br>
# CONFIG_SND_SOC_WM8962 is not set<br>
# CONFIG_SND_SOC_WM8978 is not set<br>
# CONFIG_SND_SOC_TPA6130A2 is not set<br>
CONFIG_SND_SOC_MSM_STUB=y<br>
CONFIG_SND_SOC_MSM_HDMI_DBA_CODEC_RX=y<br>
# CONFIG_SND_SIMPLE_CARD is not set<br>
# CONFIG_SOUND_PRIME is not set<br>
<br>
#<br>
# HID support<br>
#<br>
CONFIG_HID=y<br>
# CONFIG_HID_BATTERY_STRENGTH is not set<br>
CONFIG_HIDRAW=y<br>
CONFIG_UHID=y<br>
CONFIG_HID_GENERIC=y<br>
<br>
#<br>
# Special HID drivers<br>
#<br>
# CONFIG_HID_A4TECH is not set<br>
# CONFIG_HID_ACRUX is not set<br>
CONFIG_HID_APPLE=y<br>
# CONFIG_HID_APPLEIR is not set<br>
# CONFIG_HID_AUREAL is not set<br>
# CONFIG_HID_BELKIN is not set<br>
# CONFIG_HID_CHERRY is not set<br>
# CONFIG_HID_CHICONY is not set<br>
# CONFIG_HID_PRODIKEYS is not set<br>
# CONFIG_HID_CP2112 is not set<br>
# CONFIG_HID_CYPRESS is not set<br>
# CONFIG_HID_DRAGONRISE is not set<br>
# CONFIG_HID_EMS_FF is not set<br>
CONFIG_HID_ELECOM=y<br>
# CONFIG_HID_ELO is not set<br>
# CONFIG_HID_EZKEY is not set<br>
# CONFIG_HID_HOLTEK is not set<br>
# CONFIG_HID_GT683R is not set<br>
# CONFIG_HID_HUION is not set<br>
# CONFIG_HID_KEYTOUCH is not set<br>
# CONFIG_HID_KYE is not set<br>
# CONFIG_HID_UCLOGIC is not set<br>
# CONFIG_HID_WALTOP is not set<br>
# CONFIG_HID_GYRATION is not set<br>
# CONFIG_HID_ICADE is not set<br>
# CONFIG_HID_TWINHAN is not set<br>
# CONFIG_HID_KENSINGTON is not set<br>
# CONFIG_HID_LCPOWER is not set<br>
# CONFIG_HID_LENOVO is not set<br>
# CONFIG_HID_LOGITECH is not set<br>
CONFIG_HID_MAGICMOUSE=y<br>
CONFIG_HID_MICROSOFT=y<br>
# CONFIG_HID_MONTEREY is not set<br>
CONFIG_HID_MULTITOUCH=y<br>
# CONFIG_HID_NTRIG is not set<br>
# CONFIG_HID_ORTEK is not set<br>
# CONFIG_HID_PANTHERLORD is not set<br>
# CONFIG_HID_PENMOUNT is not set<br>
# CONFIG_HID_PETALYNX is not set<br>
# CONFIG_HID_PICOLCD is not set<br>
# CONFIG_HID_PRIMAX is not set<br>
# CONFIG_HID_ROCCAT is not set<br>
# CONFIG_HID_SAITEK is not set<br>
# CONFIG_HID_SAMSUNG is not set<br>
# CONFIG_HID_SONY is not set<br>
# CONFIG_HID_SPEEDLINK is not set<br>
# CONFIG_HID_STEELSERIES is not set<br>
# CONFIG_HID_SUNPLUS is not set<br>
# CONFIG_HID_RMI is not set<br>
# CONFIG_HID_GREENASIA is not set<br>
# CONFIG_HID_SMARTJOYPLUS is not set<br>
# CONFIG_HID_TIVO is not set<br>
# CONFIG_HID_TOPSEED is not set<br>
# CONFIG_HID_THINGM is not set<br>
# CONFIG_HID_THRUSTMASTER is not set<br>
# CONFIG_HID_WACOM is not set<br>
# CONFIG_HID_WIIMOTE is not set<br>
# CONFIG_HID_XINMO is not set<br>
# CONFIG_HID_ZEROPLUS is not set<br>
# CONFIG_HID_ZYDACRON is not set<br>
# CONFIG_HID_SENSOR_HUB is not set<br>
<br>
#<br>
# USB HID support<br>
#<br>
CONFIG_USB_HID=y<br>
# CONFIG_HID_PID is not set<br>
CONFIG_USB_HIDDEV=y<br>
<br>
#<br>
# I2C HID support<br>
#<br>
# CONFIG_I2C_HID is not set<br>
CONFIG_USB_OHCI_LITTLE_ENDIAN=y<br>
CONFIG_USB_SUPPORT=y<br>
CONFIG_USB_COMMON=y<br>
CONFIG_USB_ARCH_HAS_HCD=y<br>
CONFIG_USB=y<br>
CONFIG_USB_ANNOUNCE_NEW_DEVICES=y<br>
<br>
#<br>
# Miscellaneous USB options<br>
#<br>
CONFIG_USB_DEFAULT_PERSIST=y<br>
# CONFIG_USB_DYNAMIC_MINORS is not set<br>
# CONFIG_USB_OTG is not set<br>
# CONFIG_USB_OTG_WHITELIST is not set<br>
# CONFIG_USB_OTG_BLACKLIST_HUB is not set<br>
# CONFIG_USB_OTG_FSM is not set<br>
CONFIG_USB_MON=y<br>
# CONFIG_USB_WUSB_CBAF is not set<br>
<br>
#<br>
# USB Host Controller Drivers<br>
#<br>
# CONFIG_USB_C67X00_HCD is not set<br>
CONFIG_USB_XHCI_HCD=y<br>
CONFIG_USB_XHCI_PCI=y<br>
CONFIG_USB_XHCI_PLATFORM=y<br>
CONFIG_USB_EHCI_HCD=y<br>
CONFIG_USB_EHCI_ROOT_HUB_TT=y<br>
CONFIG_USB_EHCI_TT_NEWSCHED=y<br>
CONFIG_USB_EHCI_PCI=y<br>
CONFIG_USB_EHCI_MSM=y<br>
CONFIG_USB_EHCI_MSM_HSIC=y<br>
# CONFIG_USB_EHCI_HCD_PLATFORM is not set<br>
# CONFIG_USB_OXU210HP_HCD is not set<br>
# CONFIG_USB_ISP116X_HCD is not set<br>
# CONFIG_USB_ISP1760_HCD is not set<br>
# CONFIG_USB_ISP1362_HCD is not set<br>
# CONFIG_USB_FUSBH200_HCD is not set<br>
# CONFIG_USB_FOTG210_HCD is not set<br>
# CONFIG_USB_MAX3421_HCD is not set<br>
# CONFIG_USB_OHCI_HCD is not set<br>
# CONFIG_USB_UHCI_HCD is not set<br>
# CONFIG_USB_SL811_HCD is not set<br>
# CONFIG_USB_R8A66597_HCD is not set<br>
# CONFIG_USB_HCD_TEST_MODE is not set<br>
<br>
#<br>
# USB Device Class drivers<br>
#<br>
CONFIG_USB_ACM=y<br>
# CONFIG_USB_PRINTER is not set<br>
# CONFIG_USB_WDM is not set<br>
# CONFIG_USB_TMC is not set<br>
<br>
#<br>
# NOTE: USB_STORAGE depends on SCSI but BLK_DEV_SD may<br>
#<br>
<br>
#<br>
# also be needed; see USB_STORAGE Help for more info<br>
#<br>
CONFIG_USB_STORAGE=y<br>
# CONFIG_USB_STORAGE_DEBUG is not set<br>
# CONFIG_USB_STORAGE_REALTEK is not set<br>
CONFIG_USB_STORAGE_DATAFAB=y<br>
CONFIG_USB_STORAGE_FREECOM=y<br>
CONFIG_USB_STORAGE_ISD200=y<br>
CONFIG_USB_STORAGE_USBAT=y<br>
CONFIG_USB_STORAGE_SDDR09=y<br>
CONFIG_USB_STORAGE_SDDR55=y<br>
CONFIG_USB_STORAGE_JUMPSHOT=y<br>
CONFIG_USB_STORAGE_ALAUDA=y<br>
# CONFIG_USB_STORAGE_ONETOUCH is not set<br>
CONFIG_USB_STORAGE_KARMA=y<br>
CONFIG_USB_STORAGE_CYPRESS_ATACB=y<br>
# CONFIG_USB_STORAGE_ENE_UB6250 is not set<br>
# CONFIG_USB_UAS is not set<br>
<br>
#<br>
# USB Imaging devices<br>
#<br>
# CONFIG_USB_MDC800 is not set<br>
# CONFIG_USB_MICROTEK is not set<br>
# CONFIG_USBIP_CORE is not set<br>
# CONFIG_USB_MUSB_HDRC is not set<br>
CONFIG_USB_DWC3=y<br>
# CONFIG_USB_DWC3_HOST is not set<br>
# CONFIG_USB_DWC3_GADGET is not set<br>
CONFIG_USB_DWC3_DUAL_ROLE=y<br>
<br>
#<br>
# Platform Glue Driver Support<br>
#<br>
CONFIG_USB_DWC3_PCI=y<br>
CONFIG_USB_DWC3_MSM=y<br>
<br>
#<br>
# Debugging features<br>
#<br>
# CONFIG_USB_DWC3_DEBUG is not set<br>
# CONFIG_DWC3_HOST_USB3_LPM_ENABLE is not set<br>
# CONFIG_USB_DWC2 is not set<br>
# CONFIG_USB_CHIPIDEA is not set<br>
<br>
#<br>
# USB port drivers<br>
#<br>
CONFIG_USB_SERIAL=y<br>
# CONFIG_USB_SERIAL_CONSOLE is not set<br>
# CONFIG_USB_SERIAL_GENERIC is not set<br>
# CONFIG_USB_SERIAL_SIMPLE is not set<br>
# CONFIG_USB_SERIAL_AIRCABLE is not set<br>
# CONFIG_USB_SERIAL_ARK3116 is not set<br>
# CONFIG_USB_SERIAL_BELKIN is not set<br>
# CONFIG_USB_SERIAL_CH341 is not set<br>
# CONFIG_USB_SERIAL_WHITEHEAT is not set<br>
# CONFIG_USB_SERIAL_DIGI_ACCELEPORT is not set<br>
# CONFIG_USB_SERIAL_CP210X is not set<br>
# CONFIG_USB_SERIAL_CYPRESS_M8 is not set<br>
# CONFIG_USB_SERIAL_EMPEG is not set<br>
# CONFIG_USB_SERIAL_FTDI_SIO is not set<br>
# CONFIG_USB_SERIAL_VISOR is not set<br>
# CONFIG_USB_SERIAL_IPAQ is not set<br>
# CONFIG_USB_SERIAL_IR is not set<br>
# CONFIG_USB_SERIAL_EDGEPORT is not set<br>
# CONFIG_USB_SERIAL_EDGEPORT_TI is not set<br>
# CONFIG_USB_SERIAL_F81232 is not set<br>
# CONFIG_USB_SERIAL_GARMIN is not set<br>
# CONFIG_USB_SERIAL_IPW is not set<br>
# CONFIG_USB_SERIAL_IUU is not set<br>
# CONFIG_USB_SERIAL_KEYSPAN_PDA is not set<br>
# CONFIG_USB_SERIAL_KEYSPAN is not set<br>
# CONFIG_USB_SERIAL_KLSI is not set<br>
# CONFIG_USB_SERIAL_KOBIL_SCT is not set<br>
# CONFIG_USB_SERIAL_MCT_U232 is not set<br>
# CONFIG_USB_SERIAL_METRO is not set<br>
# CONFIG_USB_SERIAL_MOS7720 is not set<br>
# CONFIG_USB_SERIAL_MOS7840 is not set<br>
# CONFIG_USB_SERIAL_MXUPORT is not set<br>
# CONFIG_USB_SERIAL_NAVMAN is not set<br>
# CONFIG_USB_SERIAL_PL2303 is not set<br>
# CONFIG_USB_SERIAL_OTI6858 is not set<br>
# CONFIG_USB_SERIAL_QCAUX is not set<br>
# CONFIG_USB_SERIAL_QUALCOMM is not set<br>
# CONFIG_USB_SERIAL_SPCP8X5 is not set<br>
# CONFIG_USB_SERIAL_SAFE is not set<br>
# CONFIG_USB_SERIAL_SIERRAWIRELESS is not set<br>
# CONFIG_USB_SERIAL_SYMBOL is not set<br>
# CONFIG_USB_SERIAL_TI is not set<br>
# CONFIG_USB_SERIAL_CYBERJACK is not set<br>
# CONFIG_USB_SERIAL_XIRCOM is not set<br>
# CONFIG_USB_SERIAL_OPTION is not set<br>
# CONFIG_USB_SERIAL_OMNINET is not set<br>
# CONFIG_USB_SERIAL_OPTICON is not set<br>
# CONFIG_USB_SERIAL_XSENS_MT is not set<br>
# CONFIG_USB_SERIAL_WISHBONE is not set<br>
# CONFIG_USB_SERIAL_SSU100 is not set<br>
# CONFIG_USB_SERIAL_QT2 is not set<br>
# CONFIG_USB_SERIAL_DEBUG is not set<br>
<br>
#<br>
# USB Miscellaneous drivers<br>
#<br>
# CONFIG_USB_EMI62 is not set<br>
# CONFIG_USB_EMI26 is not set<br>
# CONFIG_USB_ADUTUX is not set<br>
# CONFIG_USB_SEVSEG is not set<br>
# CONFIG_USB_RIO500 is not set<br>
# CONFIG_USB_LEGOTOWER is not set<br>
# CONFIG_USB_LCD is not set<br>
# CONFIG_USB_LED is not set<br>
# CONFIG_USB_CYPRESS_CY7C63 is not set<br>
# CONFIG_USB_CYTHERM is not set<br>
# CONFIG_USB_IDMOUSE is not set<br>
# CONFIG_USB_FTDI_ELAN is not set<br>
# CONFIG_USB_APPLEDISPLAY is not set<br>
# CONFIG_USB_SISUSBVGA is not set<br>
# CONFIG_USB_LD is not set<br>
# CONFIG_USB_TRANCEVIBRATOR is not set<br>
# CONFIG_USB_IOWARRIOR is not set<br>
# CONFIG_USB_TEST is not set<br>
CONFIG_USB_EHSET_TEST_FIXTURE=y<br>
# CONFIG_USB_ISIGHTFW is not set<br>
# CONFIG_USB_YUREX is not set<br>
# CONFIG_USB_EZUSB_FX2 is not set<br>
# CONFIG_USB_HSIC_USB3503 is not set<br>
# CONFIG_USB_LINK_LAYER_TEST is not set<br>
<br>
#<br>
# USB Physical Layer drivers<br>
#<br>
CONFIG_USB_PHY=y<br>
# CONFIG_USB_OTG_WAKELOCK is not set<br>
CONFIG_NOP_USB_XCEIV=y<br>
# CONFIG_USB_GPIO_VBUS is not set<br>
# CONFIG_USB_ISP1301 is not set<br>
CONFIG_USB_MSM_OTG=y<br>
CONFIG_USB_MSM_HSPHY=y<br>
# CONFIG_USB_MSM_SSPHY is not set<br>
CONFIG_USB_MSM_SSPHY_QMP=y<br>
CONFIG_MSM_QUSB_PHY=y<br>
# CONFIG_USB_ULPI is not set<br>
CONFIG_DUAL_ROLE_USB_INTF=y<br>
CONFIG_USB_GADGET=y<br>
# CONFIG_USB_GADGET_DEBUG is not set<br>
CONFIG_USB_GADGET_DEBUG_FILES=y<br>
CONFIG_USB_GADGET_DEBUG_FS=y<br>
CONFIG_USB_GADGET_VBUS_DRAW=500<br>
CONFIG_USB_GADGET_STORAGE_NUM_BUFFERS=2<br>
<br>
#<br>
# USB Peripheral Controller<br>
#<br>
# CONFIG_USB_FOTG210_UDC is not set<br>
# CONFIG_USB_GR_UDC is not set<br>
# CONFIG_USB_R8A66597 is not set<br>
# CONFIG_USB_PXA27X is not set<br>
# CONFIG_USB_MV_UDC is not set<br>
# CONFIG_USB_MV_U3D is not set<br>
# CONFIG_USB_M66592 is not set<br>
# CONFIG_USB_AMD5536UDC is not set<br>
# CONFIG_USB_NET2272 is not set<br>
# CONFIG_USB_NET2280 is not set<br>
# CONFIG_USB_GOKU is not set<br>
# CONFIG_USB_EG20T is not set<br>
# CONFIG_USB_GADGET_XILINX is not set<br>
CONFIG_USB_CI13XXX_MSM=y<br>
# CONFIG_USB_CI13XXX_MSM_HSIC is not set<br>
# CONFIG_USB_DUMMY_HCD is not set<br>
CONFIG_USB_LIBCOMPOSITE=y<br>
CONFIG_USB_F_ACM=y<br>
CONFIG_USB_U_SERIAL=y<br>
CONFIG_USB_F_SERIAL=y<br>
CONFIG_USB_F_NCM=y<br>
CONFIG_USB_F_ECM=y<br>
CONFIG_USB_F_MASS_STORAGE=y<br>
CONFIG_USB_F_FS=y<br>
CONFIG_USB_F_UAC1=y<br>
CONFIG_USB_F_UAC2=y<br>
CONFIG_USB_F_UVC=y<br>
CONFIG_USB_F_AUDIO_SRC=y<br>
# CONFIG_USB_CONFIGFS is not set<br>
CONFIG_USB_G_ANDROID=y<br>
# CONFIG_USB_ANDROID_RNDIS_DWORD_ALIGNED is not set<br>
# CONFIG_USB_ZERO is not set<br>
# CONFIG_USB_AUDIO is not set<br>
# CONFIG_USB_ETH is not set<br>
# CONFIG_USB_G_NCM is not set<br>
# CONFIG_USB_GADGETFS is not set<br>
# CONFIG_USB_FUNCTIONFS is not set<br>
# CONFIG_USB_MASS_STORAGE is not set<br>
# CONFIG_USB_G_SERIAL is not set<br>
# CONFIG_USB_MIDI_GADGET is not set<br>
# CONFIG_USB_G_PRINTER is not set<br>
# CONFIG_USB_CDC_COMPOSITE is not set<br>
# CONFIG_USB_G_ACM_MS is not set<br>
# CONFIG_USB_G_MULTI is not set<br>
# CONFIG_USB_G_HID is not set<br>
# CONFIG_USB_G_DBGP is not set<br>
# CONFIG_USB_G_WEBCAM is not set<br>
# CONFIG_USB_LED_TRIG is not set<br>
# CONFIG_UWB is not set<br>
CONFIG_MMC=y<br>
# CONFIG_MMC_DEBUG is not set<br>
CONFIG_MMC_PERF_PROFILING=y<br>
CONFIG_MMC_CLKGATE=y<br>
# CONFIG_MMC_RING_BUFFER is not set<br>
# CONFIG_MMC_EMBEDDED_SDIO is not set<br>
CONFIG_MMC_PARANOID_SD_INIT=y<br>
<br>
#<br>
# MMC/SD/SDIO Card Drivers<br>
#<br>
CONFIG_MMC_BLOCK=y<br>
CONFIG_MMC_BLOCK_MINORS=32<br>
CONFIG_MMC_BLOCK_BOUNCE=y<br>
# CONFIG_MMC_BLOCK_DEFERRED_RESUME is not set<br>
# CONFIG_SDIO_UART is not set<br>
# CONFIG_MMC_TEST is not set<br>
<br>
#<br>
# MMC/SD/SDIO Host Controller Drivers<br>
#<br>
# CONFIG_MMC_ARMMMCI is not set<br>
CONFIG_MMC_SDHCI=y<br>
# CONFIG_MMC_SDHCI_PCI is not set<br>
CONFIG_MMC_SDHCI_PLTFM=y<br>
# CONFIG_MMC_SDHCI_OF_ARASAN is not set<br>
# CONFIG_MMC_SDHCI_PXAV3 is not set<br>
# CONFIG_MMC_SDHCI_PXAV2 is not set<br>
CONFIG_MMC_SDHCI_MSM=y<br>
CONFIG_MMC_SDHCI_MSM_ICE=y<br>
# CONFIG_MMC_TIFM_SD is not set<br>
# CONFIG_MMC_SPI is not set<br>
# CONFIG_MMC_CB710 is not set<br>
# CONFIG_MMC_VIA_SDMMC is not set<br>
# CONFIG_MMC_VUB300 is not set<br>
# CONFIG_MMC_USHC is not set<br>
# CONFIG_MMC_USDHI6ROL0 is not set<br>
CONFIG_MMC_CQ_HCI=y<br>
# CONFIG_MEMSTICK is not set<br>
CONFIG_NEW_LEDS=y<br>
CONFIG_LEDS_CLASS=y<br>
<br>
#<br>
# LED drivers<br>
#<br>
# CONFIG_LEDS_LM3530 is not set<br>
# CONFIG_LEDS_LM3642 is not set<br>
# CONFIG_LEDS_PCA9532 is not set<br>
# CONFIG_LEDS_GPIO is not set<br>
# CONFIG_LEDS_LP3944 is not set<br>
# CONFIG_LEDS_LP5521 is not set<br>
# CONFIG_LEDS_LP5523 is not set<br>
# CONFIG_LEDS_LP5562 is not set<br>
# CONFIG_LEDS_LP8501 is not set<br>
# CONFIG_LEDS_PCA955X is not set<br>
# CONFIG_LEDS_PCA963X is not set<br>
# CONFIG_LEDS_DAC124S085 is not set<br>
# CONFIG_LEDS_PWM is not set<br>
# CONFIG_LEDS_REGULATOR is not set<br>
# CONFIG_LEDS_BD2802 is not set<br>
# CONFIG_LEDS_INTEL_SS4200 is not set<br>
# CONFIG_LEDS_LT3593 is not set<br>
# CONFIG_LEDS_TCA6507 is not set<br>
# CONFIG_LEDS_LM355x is not set<br>
<br>
#<br>
# LED driver for blink(1) USB RGB LED is under Special HID drivers (HID_THINGM)<br>
#<br>
# CONFIG_LEDS_BLINKM is not set<br>
CONFIG_LEDS_QPNP=y<br>
CONFIG_LEDS_QPNP_FLASH=y<br>
CONFIG_LEDS_QPNP_WLED=y<br>
CONFIG_LEDS_AW2013=y<br>
<br>
#<br>
# LED Triggers<br>
#<br>
CONFIG_LEDS_TRIGGERS=y<br>
# CONFIG_LEDS_TRIGGER_TIMER is not set<br>
# CONFIG_LEDS_TRIGGER_ONESHOT is not set<br>
# CONFIG_LEDS_TRIGGER_HEARTBEAT is not set<br>
# CONFIG_LEDS_TRIGGER_BACKLIGHT is not set<br>
# CONFIG_LEDS_TRIGGER_CPU is not set<br>
# CONFIG_LEDS_TRIGGER_GPIO is not set<br>
# CONFIG_LEDS_TRIGGER_DEFAULT_ON is not set<br>
<br>
#<br>
# iptables trigger is under Netfilter config (LED target)<br>
#<br>
# CONFIG_LEDS_TRIGGER_TRANSIENT is not set<br>
# CONFIG_LEDS_TRIGGER_CAMERA is not set<br>
CONFIG_SWITCH=y<br>
# CONFIG_SWITCH_GPIO is not set<br>
# CONFIG_ACCESSIBILITY is not set<br>
# CONFIG_INFINIBAND is not set<br>
CONFIG_EDAC_SUPPORT=y<br>
CONFIG_EDAC=y<br>
CONFIG_EDAC_LEGACY_SYSFS=y<br>
# CONFIG_EDAC_DEBUG is not set<br>
CONFIG_EDAC_MM_EDAC=y<br>
CONFIG_EDAC_CORTEX_ARM64=y<br>
# CONFIG_EDAC_CORTEX_ARM64_PANIC_ON_CE is not set<br>
CONFIG_EDAC_CORTEX_ARM64_DBE_IRQ_ONLY=y<br>
CONFIG_EDAC_CORTEX_ARM64_PANIC_ON_UE=y<br>
CONFIG_RTC_LIB=y<br>
CONFIG_RTC_CLASS=y<br>
CONFIG_RTC_HCTOSYS=y<br>
CONFIG_RTC_SYSTOHC=y<br>
CONFIG_RTC_HCTOSYS_DEVICE="rtc0"<br>
# CONFIG_RTC_DEBUG is not set<br>
<br>
#<br>
# RTC interfaces<br>
#<br>
CONFIG_RTC_INTF_SYSFS=y<br>
CONFIG_RTC_INTF_PROC=y<br>
CONFIG_RTC_INTF_DEV=y<br>
# CONFIG_RTC_INTF_DEV_UIE_EMUL is not set<br>
# CONFIG_RTC_DRV_TEST is not set<br>
<br>
#<br>
# I2C RTC drivers<br>
#<br>
# CONFIG_RTC_DRV_DS1307 is not set<br>
# CONFIG_RTC_DRV_DS1374 is not set<br>
# CONFIG_RTC_DRV_DS1672 is not set<br>
# CONFIG_RTC_DRV_DS3232 is not set<br>
# CONFIG_RTC_DRV_HYM8563 is not set<br>
# CONFIG_RTC_DRV_MAX6900 is not set<br>
# CONFIG_RTC_DRV_RS5C372 is not set<br>
# CONFIG_RTC_DRV_ISL1208 is not set<br>
# CONFIG_RTC_DRV_ISL12022 is not set<br>
# CONFIG_RTC_DRV_ISL12057 is not set<br>
# CONFIG_RTC_DRV_X1205 is not set<br>
# CONFIG_RTC_DRV_PCF2127 is not set<br>
# CONFIG_RTC_DRV_PCF8523 is not set<br>
# CONFIG_RTC_DRV_PCF8563 is not set<br>
# CONFIG_RTC_DRV_PCF85063 is not set<br>
# CONFIG_RTC_DRV_PCF8583 is not set<br>
# CONFIG_RTC_DRV_M41T80 is not set<br>
# CONFIG_RTC_DRV_BQ32K is not set<br>
# CONFIG_RTC_DRV_S35390A is not set<br>
# CONFIG_RTC_DRV_FM3130 is not set<br>
# CONFIG_RTC_DRV_RX8581 is not set<br>
# CONFIG_RTC_DRV_RX8025 is not set<br>
# CONFIG_RTC_DRV_EM3027 is not set<br>
# CONFIG_RTC_DRV_RV3029C2 is not set<br>
<br>
#<br>
# SPI RTC drivers<br>
#<br>
# CONFIG_RTC_DRV_M41T93 is not set<br>
# CONFIG_RTC_DRV_M41T94 is not set<br>
# CONFIG_RTC_DRV_DS1305 is not set<br>
# CONFIG_RTC_DRV_DS1343 is not set<br>
# CONFIG_RTC_DRV_DS1347 is not set<br>
# CONFIG_RTC_DRV_DS1390 is not set<br>
# CONFIG_RTC_DRV_MAX6902 is not set<br>
# CONFIG_RTC_DRV_R9701 is not set<br>
# CONFIG_RTC_DRV_RS5C348 is not set<br>
# CONFIG_RTC_DRV_DS3234 is not set<br>
# CONFIG_RTC_DRV_PCF2123 is not set<br>
# CONFIG_RTC_DRV_RX4581 is not set<br>
# CONFIG_RTC_DRV_MCP795 is not set<br>
<br>
#<br>
# Platform RTC drivers<br>
#<br>
# CONFIG_RTC_DRV_DS1286 is not set<br>
# CONFIG_RTC_DRV_DS1511 is not set<br>
# CONFIG_RTC_DRV_DS1553 is not set<br>
# CONFIG_RTC_DRV_DS1742 is not set<br>
# CONFIG_RTC_DRV_DS2404 is not set<br>
# CONFIG_RTC_DRV_EFI is not set<br>
# CONFIG_RTC_DRV_STK17TA8 is not set<br>
# CONFIG_RTC_DRV_M48T86 is not set<br>
# CONFIG_RTC_DRV_M48T35 is not set<br>
# CONFIG_RTC_DRV_M48T59 is not set<br>
# CONFIG_RTC_DRV_MSM6242 is not set<br>
# CONFIG_RTC_DRV_BQ4802 is not set<br>
# CONFIG_RTC_DRV_RP5C01 is not set<br>
# CONFIG_RTC_DRV_V3020 is not set<br>
<br>
#<br>
# on-CPU RTC drivers<br>
#<br>
# CONFIG_RTC_DRV_PL030 is not set<br>
# CONFIG_RTC_DRV_PL031 is not set<br>
# CONFIG_RTC_DRV_SNVS is not set<br>
CONFIG_RTC_DRV_QPNP=y<br>
# CONFIG_RTC_DRV_XGENE is not set<br>
<br>
#<br>
# HID Sensor RTC drivers<br>
#<br>
# CONFIG_RTC_DRV_HID_SENSOR_TIME is not set<br>
# CONFIG_ESOC is not set<br>
CONFIG_DMADEVICES=y<br>
# CONFIG_DMADEVICES_DEBUG is not set<br>
<br>
#<br>
# DMA Devices<br>
#<br>
# CONFIG_AMBA_PL08X is not set<br>
# CONFIG_DW_DMAC_CORE is not set<br>
# CONFIG_DW_DMAC is not set<br>
# CONFIG_DW_DMAC_PCI is not set<br>
CONFIG_QCOM_SPS_DMA=y<br>
# CONFIG_PL330_DMA is not set<br>
# CONFIG_FSL_EDMA is not set<br>
CONFIG_DMA_ENGINE=y<br>
CONFIG_DMA_OF=y<br>
<br>
#<br>
# DMA Clients<br>
#<br>
# CONFIG_ASYNC_TX_DMA is not set<br>
# CONFIG_DMATEST is not set<br>
# CONFIG_AUXDISPLAY is not set<br>
CONFIG_UIO=y<br>
# CONFIG_UIO_CIF is not set<br>
# CONFIG_UIO_PDRV_GENIRQ is not set<br>
# CONFIG_UIO_DMEM_GENIRQ is not set<br>
# CONFIG_UIO_AEC is not set<br>
# CONFIG_UIO_SERCOS3 is not set<br>
# CONFIG_UIO_PCI_GENERIC is not set<br>
# CONFIG_UIO_NETX is not set<br>
# CONFIG_UIO_MF624 is not set<br>
CONFIG_UIO_MSM_SHAREDMEM=y<br>
# CONFIG_VFIO is not set<br>
# CONFIG_VIRT_DRIVERS is not set<br>
<br>
#<br>
# Virtio drivers<br>
#<br>
# CONFIG_VIRTIO_PCI is not set<br>
# CONFIG_VIRTIO_MMIO is not set<br>
<br>
#<br>
# Microsoft Hyper-V guest support<br>
#<br>
CONFIG_STAGING=y<br>
# CONFIG_PRISM2_USB is not set<br>
# CONFIG_COMEDI is not set<br>
# CONFIG_RTL8192U is not set<br>
# CONFIG_RTLLIB is not set<br>
# CONFIG_R8712U is not set<br>
# CONFIG_R8188EU is not set<br>
# CONFIG_R8723AU is not set<br>
# CONFIG_RTS5208 is not set<br>
# CONFIG_LINE6_USB is not set<br>
# CONFIG_VT6655 is not set<br>
CONFIG_SNAPPY_COMPRESS=y<br>
CONFIG_SNAPPY_DECOMPRESS=y<br>
# CONFIG_FB_XGI is not set<br>
# CONFIG_FT1000 is not set<br>
<br>
#<br>
# Speakup console speech<br>
#<br>
# CONFIG_SPEAKUP is not set<br>
# CONFIG_TOUCHSCREEN_CLEARPAD_TM1217 is not set<br>
# CONFIG_STAGING_MEDIA is not set<br>
<br>
#<br>
# Android<br>
#<br>
CONFIG_ANDROID=y<br>
CONFIG_ANDROID_BINDER_IPC=y<br>
CONFIG_ANDROID_BINDER_DEVICES="binder,hwbinder,vndbinder"<br>
CONFIG_ASHMEM=y<br>
CONFIG_ANDROID_LOGGER=y<br>
CONFIG_ANDROID_TIMED_OUTPUT=y<br>
CONFIG_ANDROID_TIMED_GPIO=y<br>
# CONFIG_ANDROID_LOW_MEMORY_KILLER is not set<br>
CONFIG_SYNC=y<br>
CONFIG_SW_SYNC=y<br>
CONFIG_SW_SYNC_USER=y<br>
CONFIG_ONESHOT_SYNC=y<br>
# CONFIG_ONESHOT_SYNC_USER is not set<br>
CONFIG_ION=y<br>
# CONFIG_ION_TEST is not set<br>
# CONFIG_ION_DUMMY is not set<br>
CONFIG_ION_MSM=y<br>
# CONFIG_ALLOC_BUFFERS_IN_4K_CHUNKS is not set<br>
# CONFIG_FIQ_DEBUGGER is not set<br>
# CONFIG_FIQ_WATCHDOG is not set<br>
# CONFIG_USB_WPAN_HCD is not set<br>
# CONFIG_WIMAX_GDM72XX is not set<br>
# CONFIG_LTE_GDM724X is not set<br>
# CONFIG_LUSTRE_FS is not set<br>
# CONFIG_DGNC is not set<br>
# CONFIG_DGAP is not set<br>
# CONFIG_GS_FPGABOOT is not set<br>
<br>
#<br>
# Qualcomm Atheros Prima WLAN module<br>
#<br>
# CONFIG_PRIMA_WLAN is not set<br>
CONFIG_PRONTO_WLAN=m<br>
# CONFIG_PRIMA_WLAN_BTAMP is not set<br>
CONFIG_PRIMA_WLAN_LFR=y<br>
CONFIG_PRIMA_WLAN_OKC=y<br>
CONFIG_PRIMA_WLAN_11AC_HIGH_TP=y<br>
CONFIG_WLAN_FEATURE_11W=y<br>
CONFIG_QCOM_VOWIFI_11R=y<br>
CONFIG_ENABLE_LINUX_REG=y<br>
CONFIG_WLAN_OFFLOAD_PACKETS=y<br>
CONFIG_QCOM_TDLS=y<br>
# CONFIG_GOLDFISH is not set<br>
<br>
#<br>
# Qualcomm MSM specific device drivers<br>
#<br>
CONFIG_MSM_AVTIMER=y<br>
CONFIG_MSM_BUS_SCALING=y<br>
CONFIG_BUS_TOPOLOGY_ADHOC=y<br>
# CONFIG_DEBUG_BUS_VOTER is not set<br>
CONFIG_QPNP_POWER_ON=y<br>
CONFIG_QPNP_REVID=y<br>
CONFIG_QPNP_COINCELL=y<br>
CONFIG_SPS=y<br>
# CONFIG_EP_PCIE is not set<br>
CONFIG_USB_BAM=y<br>
# CONFIG_SPS_SUPPORT_BAMDMA is not set<br>
CONFIG_SPS_SUPPORT_NDP_BAM=y<br>
# CONFIG_QPNP_VIBRATOR is not set<br>
CONFIG_IPA=y<br>
# CONFIG_IPA3 is not set<br>
# CONFIG_GSI is not set<br>
CONFIG_RMNET_IPA=y<br>
# CONFIG_SSM is not set<br>
# CONFIG_MSM_MHI is not set<br>
# CONFIG_PFT is not set<br>
# CONFIG_I2C_MSM_PROF_DBG is not set<br>
# CONFIG_SEEMP_CORE is not set<br>
CONFIG_QPNP_HAPTIC=y<br>
# CONFIG_GPIO_USB_DETECT is not set<br>
CONFIG_MSM_11AD=y<br>
# CONFIG_BW_MONITOR is not set<br>
CONFIG_MSM_SPMI=y<br>
CONFIG_MSM_SPMI_PMIC_ARB=y<br>
CONFIG_MSM_QPNP_INT=y<br>
# CONFIG_MSM_SPMI_DEBUGFS_RO is not set<br>
CONFIG_MACH_XIAOMI_MSM8953=y<br>
<br>
#<br>
# Xiaomi board selection<br>
#<br>
CONFIG_MACH_XIAOMI_MIDO=y<br>
CONFIG_CLKDEV_LOOKUP=y<br>
CONFIG_HAVE_CLK_PREPARE=y<br>
CONFIG_MSM_CLK_CONTROLLER_V2=y<br>
CONFIG_MSM_MDSS_PLL=y<br>
CONFIG_HWSPINLOCK=y<br>
<br>
#<br>
# Hardware Spinlock drivers<br>
#<br>
CONFIG_REMOTE_SPINLOCK_MSM=y<br>
<br>
#<br>
# Clock Source drivers<br>
#<br>
CONFIG_CLKSRC_OF=y<br>
CONFIG_ARM_ARCH_TIMER=y<br>
CONFIG_ARM_ARCH_TIMER_EVTSTREAM=y<br>
# CONFIG_ARM_ARCH_TIMER_VCT_ACCESS is not set<br>
# CONFIG_ATMEL_PIT is not set<br>
# CONFIG_SH_TIMER_CMT is not set<br>
# CONFIG_SH_TIMER_MTU2 is not set<br>
# CONFIG_SH_TIMER_TMU is not set<br>
# CONFIG_EM_TIMER_STI is not set<br>
# CONFIG_CLKSRC_VERSATILE is not set<br>
# CONFIG_MAILBOX is not set<br>
CONFIG_IOMMU_API=y<br>
CONFIG_IOMMU_SUPPORT=y<br>
<br>
#<br>
# Generic IOMMU Pagetable Support<br>
#<br>
CONFIG_IOMMU_IO_PGTABLE=y<br>
CONFIG_IOMMU_IO_PGTABLE_LPAE=y<br>
# CONFIG_IOMMU_IO_PGTABLE_LPAE_SELFTEST is not set<br>
# CONFIG_IOMMU_IO_PGTABLE_FAST is not set<br>
CONFIG_OF_IOMMU=y<br>
CONFIG_MSM_IOMMU=y<br>
CONFIG_MSM_IOMMU_V1=y<br>
# CONFIG_IOMMU_LPAE is not set<br>
# CONFIG_IOMMU_AARCH64 is not set<br>
# CONFIG_MSM_IOMMU_VBIF_CHECK is not set<br>
# CONFIG_IOMMU_NON_SECURE is not set<br>
# CONFIG_IOMMU_FORCE_4K_MAPPINGS is not set<br>
CONFIG_ARM_SMMU=y<br>
CONFIG_IOMMU_DEBUG=y<br>
# CONFIG_IOMMU_DEBUG_TRACKING is not set<br>
CONFIG_IOMMU_TESTS=y<br>
<br>
#<br>
# Remoteproc drivers<br>
#<br>
# CONFIG_STE_MODEM_RPROC is not set<br>
<br>
#<br>
# Rpmsg drivers<br>
#<br>
<br>
#<br>
# SOC (System On Chip) specific Drivers<br>
#<br>
CONFIG_CP_ACCESS64=y<br>
# CONFIG_MSM_INRUSH_CURRENT_MITIGATION is not set<br>
CONFIG_MSM_QDSP6_APRV2=y<br>
# CONFIG_MSM_GLADIATOR_ERP is not set<br>
# CONFIG_MSM_GLADIATOR_ERP_V2 is not set<br>
# CONFIG_MSM_QDSP6_APRV3 is not set<br>
# CONFIG_MSM_QDSP6_APRV2_GLINK is not set<br>
# CONFIG_MSM_QDSP6_APRV3_GLINK is not set<br>
CONFIG_MSM_ADSP_LOADER=y<br>
# CONFIG_MSM_MEMORY_DUMP is not set<br>
CONFIG_MSM_MEMORY_DUMP_V2=y<br>
# CONFIG_MSM_DEBUG_LAR_UNLOCK is not set<br>
# CONFIG_MSM_JTAG is not set<br>
# CONFIG_MSM_JTAG_MM is not set<br>
# CONFIG_MSM_JTAGV8 is not set<br>
CONFIG_MSM_BOOT_STATS=y<br>
# CONFIG_MSM_BOOT_TIME_MARKER is not set<br>
CONFIG_MSM_CPUSS_DUMP=y<br>
CONFIG_MSM_COMMON_LOG=y<br>
CONFIG_MSM_DDR_HEALTH=y<br>
# CONFIG_MSM_HYP_DEBUG is not set<br>
CONFIG_MSM_WATCHDOG_V2=y<br>
CONFIG_MSM_FORCE_WDOG_BITE_ON_PANIC=y<br>
# CONFIG_MSM_CORE_HANG_DETECT is not set<br>
# CONFIG_MSM_GLADIATOR_HANG_DETECT is not set<br>
CONFIG_MSM_CPU_PWR_CTL=y<br>
# CONFIG_MSM_L2_IA_DEBUG is not set<br>
CONFIG_MSM_RPM_SMD=y<br>
CONFIG_MSM_RPM_RBCPR_STATS_V2_LOG=y<br>
CONFIG_MSM_RPM_LOG=y<br>
CONFIG_MSM_RPM_STATS_LOG=y<br>
CONFIG_MSM_RUN_QUEUE_STATS=y<br>
CONFIG_MSM_SCM=y<br>
CONFIG_MSM_SCM_XPU=y<br>
CONFIG_MSM_XPU_ERR_FATAL=y<br>
# CONFIG_MSM_XPU_ERR_NONFATAL is not set<br>
# CONFIG_MSM_SCM_ERRATA is not set<br>
# CONFIG_MSM_PFE_WA is not set<br>
CONFIG_MSM_MPM_OF=y<br>
CONFIG_MSM_SMEM=y<br>
CONFIG_MSM_SMD=y<br>
CONFIG_MSM_SMD_DEBUG=y<br>
CONFIG_MSM_GLINK=y<br>
CONFIG_MSM_GLINK_LOOPBACK_SERVER=y<br>
CONFIG_MSM_GLINK_SMD_XPRT=y<br>
CONFIG_MSM_GLINK_SMEM_NATIVE_XPRT=y<br>
CONFIG_MSM_SMEM_LOGGING=y<br>
CONFIG_MSM_SMP2P=y<br>
CONFIG_MSM_SMP2P_TEST=y<br>
CONFIG_MSM_SPM=y<br>
CONFIG_MSM_L2_SPM=y<br>
CONFIG_MSM_QMI_INTERFACE=y<br>
# CONFIG_MSM_DCC is not set<br>
# CONFIG_MSM_HVC is not set<br>
CONFIG_MSM_IPC_ROUTER_SMD_XPRT=y<br>
CONFIG_MSM_EVENT_TIMER=y<br>
# CONFIG_MSM_SYSMON_GLINK_COMM is not set<br>
# CONFIG_MSM_IPC_ROUTER_GLINK_XPRT is not set<br>
# CONFIG_MSM_SYSTEM_HEALTH_MONITOR is not set<br>
# CONFIG_MSM_GLINK_PKT is not set<br>
CONFIG_MSM_TZ_SMMU=y<br>
CONFIG_MSM_SUBSYSTEM_RESTART=y<br>
CONFIG_MSM_SYSMON_COMM=y<br>
CONFIG_MSM_PIL=y<br>
CONFIG_MSM_PIL_SSR_GENERIC=y<br>
CONFIG_MSM_PIL_MSS_QDSP6V5=y<br>
# CONFIG_MSM_SHARED_HEAP_ACCESS is not set<br>
# CONFIG_TRACER_PKT is not set<br>
CONFIG_MSM_SECURE_BUFFER=y<br>
CONFIG_ICNSS=y<br>
CONFIG_MSM_BAM_DMUX=y<br>
CONFIG_MSM_PERFORMANCE=y<br>
CONFIG_MSM_PERFORMANCE_HOTPLUG_ON=y<br>
# CONFIG_MSM_POWER is not set<br>
# CONFIG_MSM_SERVICE_LOCATOR is not set<br>
# CONFIG_MSM_QBT1000 is not set<br>
# CONFIG_MSM_PACMAN is not set<br>
# CONFIG_MSM_REMOTEQDSS is not set<br>
# CONFIG_QCOM_SMCINVOKE is not set<br>
CONFIG_SERIAL_NUM=y<br>
CONFIG_STATE_NOTIFIER=y<br>
CONFIG_MEM_SHARE_QMI_SERVICE=y<br>
# CONFIG_SOC_TI is not set<br>
CONFIG_PM_DEVFREQ=y<br>
<br>
#<br>
# DEVFREQ Governors<br>
#<br>
CONFIG_DEVFREQ_GOV_SIMPLE_ONDEMAND=y<br>
CONFIG_DEVFREQ_GOV_PERFORMANCE=y<br>
CONFIG_DEVFREQ_GOV_POWERSAVE=y<br>
CONFIG_DEVFREQ_GOV_USERSPACE=y<br>
CONFIG_DEVFREQ_GOV_CPUFREQ=y<br>
CONFIG_DEVFREQ_GOV_MSM_ADRENO_TZ=y<br>
# CONFIG_SIMPLE_GPU_ALGORITHM is not set<br>
CONFIG_MSM_BIMC_BWMON=y<br>
CONFIG_DEVFREQ_GOV_MSM_GPUBW_MON=y<br>
# CONFIG_ARMBW_HWMON is not set<br>
CONFIG_ARM_MEMLAT_MON=y<br>
CONFIG_MSMCCI_HWMON=y<br>
CONFIG_MSM_M4M_HWMON=y<br>
CONFIG_DEVFREQ_GOV_MSM_BW_HWMON=y<br>
CONFIG_DEVFREQ_GOV_MSM_CACHE_HWMON=y<br>
CONFIG_DEVFREQ_GOV_SPDM_HYP=y<br>
CONFIG_DEVFREQ_GOV_MEMLAT=y<br>
CONFIG_ADRENO_IDLER=y<br>
<br>
#<br>
# DEVFREQ Drivers<br>
#<br>
CONFIG_DEVFREQ_SIMPLE_DEV=y<br>
CONFIG_MSM_DEVFREQ_DEVBW=y<br>
CONFIG_SPDM_SCM=y<br>
CONFIG_DEVFREQ_SPDM=y<br>
# CONFIG_EXTCON is not set<br>
# CONFIG_MEMORY is not set<br>
# CONFIG_IIO is not set<br>
# CONFIG_VME_BUS is not set<br>
CONFIG_PWM=y<br>
CONFIG_PWM_SYSFS=y<br>
# CONFIG_PWM_FSL_FTM is not set<br>
# CONFIG_PWM_PCA9685 is not set<br>
CONFIG_PWM_QPNP=y<br>
CONFIG_IRQCHIP=y<br>
CONFIG_ARM_GIC=y<br>
CONFIG_ARM_GIC_V2M=y<br>
CONFIG_ARM_GIC_V3=y<br>
CONFIG_ARM_GIC_PANIC_HANDLER=y<br>
CONFIG_ARM_GIC_V3_ITS=y<br>
CONFIG_ARM_GIC_V3_ACL=y<br>
# CONFIG_ARM_GIC_V3_NO_ACCESS_CONTROL is not set<br>
CONFIG_MSM_SHOW_RESUME_IRQ=y<br>
CONFIG_MSM_IRQ=y<br>
# CONFIG_IPACK_BUS is not set<br>
# CONFIG_RESET_CONTROLLER is not set<br>
# CONFIG_FMC is not set<br>
CONFIG_CORESIGHT=y<br>
CONFIG_CORESIGHT_EVENT=y<br>
CONFIG_HAVE_CORESIGHT_SINK=y<br>
CONFIG_CORESIGHT_FUSE=y<br>
CONFIG_CORESIGHT_CTI=y<br>
CONFIG_CORESIGHT_CTI_SAVE_DISABLE=y<br>
CONFIG_CORESIGHT_CSR=y<br>
CONFIG_CORESIGHT_TMC=y<br>
CONFIG_CORESIGHT_TPIU=y<br>
CONFIG_CORESIGHT_FUNNEL=y<br>
CONFIG_CORESIGHT_REPLICATOR=y<br>
# CONFIG_CORESIGHT_TPDA is not set<br>
# CONFIG_CORESIGHT_TPDM is not set<br>
# CONFIG_CORESIGHT_DBGUI is not set<br>
CONFIG_CORESIGHT_STM=y<br>
# CONFIG_CORESIGHT_STM_DEFAULT_ENABLE is not set<br>
CONFIG_CORESIGHT_HWEVENT=y<br>
# CONFIG_CORESIGHT_ETM is not set<br>
# CONFIG_CORESIGHT_ETMV4 is not set<br>
# CONFIG_CORESIGHT_REMOTE_ETM is not set<br>
# CONFIG_CORESIGHT_QPDI is not set<br>
<br>
#<br>
# PHY Subsystem<br>
#<br>
CONFIG_GENERIC_PHY=y<br>
# CONFIG_BCM_KONA_USB2_PHY is not set<br>
# CONFIG_PHY_XGENE is not set<br>
CONFIG_PHY_QCOM_UFS=y<br>
# CONFIG_POWERCAP is not set<br>
# CONFIG_MCB is not set<br>
CONFIG_RAS=y<br>
# CONFIG_THUNDERBOLT is not set<br>
CONFIG_SENSORS_SSC=y<br>
<br>
#<br>
# Firmware Drivers<br>
#<br>
# CONFIG_FIRMWARE_MEMMAP is not set<br>
CONFIG_DMIID=y<br>
# CONFIG_DMI_SYSFS is not set<br>
<br>
#<br>
# EFI (Extensible Firmware Interface) Support<br>
#<br>
# CONFIG_EFI_VARS is not set<br>
CONFIG_EFI_PARAMS_FROM_FDT=y<br>
CONFIG_EFI_RUNTIME_WRAPPERS=y<br>
CONFIG_EFI_ARMSTUB=y<br>
CONFIG_MSM_TZ_LOG=y<br>
# CONFIG_BIF is not set<br>
<br>
#<br>
# Firmware Drivers<br>
#<br>
<br>
#<br>
# EFI (Extensible Firmware Interface) Support<br>
#<br>
<br>
#<br>
# File systems<br>
#<br>
CONFIG_DCACHE_WORD_ACCESS=y<br>
CONFIG_EXT2_FS=y<br>
CONFIG_EXT2_FS_XATTR=y<br>
# CONFIG_EXT2_FS_POSIX_ACL is not set<br>
# CONFIG_EXT2_FS_SECURITY is not set<br>
# CONFIG_EXT2_FS_XIP is not set<br>
CONFIG_EXT3_FS=y<br>
# CONFIG_EXT3_DEFAULTS_TO_ORDERED is not set<br>
CONFIG_EXT3_FS_XATTR=y<br>
# CONFIG_EXT3_FS_POSIX_ACL is not set<br>
# CONFIG_EXT3_FS_SECURITY is not set<br>
CONFIG_EXT4_FS=y<br>
# CONFIG_EXT4_FS_POSIX_ACL is not set<br>
CONFIG_EXT4_FS_SECURITY=y<br>
# CONFIG_EXT4_DEBUG is not set<br>
CONFIG_JBD=y<br>
# CONFIG_JBD_DEBUG is not set<br>
CONFIG_JBD2=y<br>
# CONFIG_JBD2_DEBUG is not set<br>
CONFIG_FS_MBCACHE=y<br>
# CONFIG_REISERFS_FS is not set<br>
# CONFIG_JFS_FS is not set<br>
# CONFIG_XFS_FS is not set<br>
# CONFIG_GFS2_FS is not set<br>
# CONFIG_OCFS2_FS is not set<br>
# CONFIG_BTRFS_FS is not set<br>
# CONFIG_NILFS2_FS is not set<br>
CONFIG_FS_POSIX_ACL=y<br>
CONFIG_EXPORTFS=y<br>
CONFIG_FILE_LOCKING=y<br>
CONFIG_FS_ENCRYPTION=y<br>
CONFIG_FSNOTIFY=y<br>
CONFIG_DNOTIFY=y<br>
CONFIG_INOTIFY_USER=y<br>
CONFIG_FANOTIFY=y<br>
# CONFIG_FANOTIFY_ACCESS_PERMISSIONS is not set<br>
CONFIG_QUOTA=y<br>
# CONFIG_QUOTA_NETLINK_INTERFACE is not set<br>
# CONFIG_PRINT_QUOTA_WARNING is not set<br>
# CONFIG_QUOTA_DEBUG is not set<br>
# CONFIG_QFMT_V1 is not set<br>
# CONFIG_QFMT_V2 is not set<br>
CONFIG_QUOTACTL=y<br>
CONFIG_AUTOFS4_FS=m<br>
CONFIG_FUSE_FS=y<br>
# CONFIG_CUSE is not set<br>
CONFIG_OVERLAY_FS=m<br>
<br>
#<br>
# Caches<br>
#<br>
# CONFIG_FSCACHE is not set<br>
<br>
#<br>
# CD-ROM/DVD Filesystems<br>
#<br>
# CONFIG_ISO9660_FS is not set<br>
# CONFIG_UDF_FS is not set<br>
<br>
#<br>
# DOS/FAT/NT Filesystems<br>
#<br>
CONFIG_FAT_FS=y<br>
CONFIG_MSDOS_FS=y<br>
CONFIG_VFAT_FS=y<br>
CONFIG_FAT_DEFAULT_CODEPAGE=437<br>
CONFIG_FAT_DEFAULT_IOCHARSET="iso8859-1"<br>
CONFIG_EXFAT_FS=y<br>
CONFIG_EXFAT_DISCARD=y<br>
# CONFIG_EXFAT_DELAYED_SYNC is not set<br>
# CONFIG_EXFAT_KERNEL_DEBUG is not set<br>
# CONFIG_EXFAT_DEBUG_MSG is not set<br>
CONFIG_EXFAT_DEFAULT_CODEPAGE=437<br>
CONFIG_EXFAT_DEFAULT_IOCHARSET="utf8"<br>
# CONFIG_NTFS_FS is not set<br>
<br>
#<br>
# Pseudo filesystems<br>
#<br>
CONFIG_PROC_FS=y<br>
# CONFIG_PROC_KCORE is not set<br>
CONFIG_PROC_SYSCTL=y<br>
CONFIG_PROC_PAGE_MONITOR=y<br>
CONFIG_KERNFS=y<br>
CONFIG_SYSFS=y<br>
CONFIG_TMPFS=y<br>
CONFIG_TMPFS_POSIX_ACL=y<br>
CONFIG_TMPFS_XATTR=y<br>
# CONFIG_HUGETLBFS is not set<br>
# CONFIG_HUGETLB_PAGE is not set<br>
CONFIG_CONFIGFS_FS=y<br>
CONFIG_MISC_FILESYSTEMS=y<br>
# CONFIG_ADFS_FS is not set<br>
# CONFIG_AFFS_FS is not set<br>
CONFIG_ECRYPT_FS=y<br>
# CONFIG_ECRYPT_FS_MESSAGING is not set<br>
# CONFIG_SDCARD_FS is not set<br>
# CONFIG_HFS_FS is not set<br>
# CONFIG_HFSPLUS_FS is not set<br>
# CONFIG_BEFS_FS is not set<br>
# CONFIG_BFS_FS is not set<br>
# CONFIG_EFS_FS is not set<br>
# CONFIG_LOGFS is not set<br>
# CONFIG_CRAMFS is not set<br>
CONFIG_SQUASHFS=y<br>
CONFIG_SQUASHFS_FILE_CACHE=y<br>
# CONFIG_SQUASHFS_FILE_DIRECT is not set<br>
CONFIG_SQUASHFS_DECOMP_SINGLE=y<br>
# CONFIG_SQUASHFS_DECOMP_MULTI is not set<br>
# CONFIG_SQUASHFS_DECOMP_MULTI_PERCPU is not set<br>
# CONFIG_SQUASHFS_XATTR is not set<br>
CONFIG_SQUASHFS_ZLIB=y<br>
# CONFIG_SQUASHFS_LZ4 is not set<br>
# CONFIG_SQUASHFS_LZO is not set<br>
CONFIG_SQUASHFS_XZ=y<br>
# CONFIG_SQUASHFS_4K_DEVBLK_SIZE is not set<br>
# CONFIG_SQUASHFS_EMBEDDED is not set<br>
CONFIG_SQUASHFS_FRAGMENT_CACHE_SIZE=3<br>
# CONFIG_VXFS_FS is not set<br>
# CONFIG_MINIX_FS is not set<br>
# CONFIG_OMFS_FS is not set<br>
# CONFIG_HPFS_FS is not set<br>
# CONFIG_QNX4FS_FS is not set<br>
# CONFIG_QNX6FS_FS is not set<br>
# CONFIG_ROMFS_FS is not set<br>
CONFIG_PSTORE=y<br>
CONFIG_PSTORE_CONSOLE=y<br>
# CONFIG_PSTORE_PMSG is not set<br>
CONFIG_PSTORE_FTRACE=y<br>
CONFIG_PSTORE_RAM=y<br>
# CONFIG_SYSV_FS is not set<br>
# CONFIG_UFS_FS is not set<br>
CONFIG_F2FS_FS=y<br>
CONFIG_F2FS_STAT_FS=y<br>
CONFIG_F2FS_FS_XATTR=y<br>
CONFIG_F2FS_FS_POSIX_ACL=y<br>
CONFIG_F2FS_FS_SECURITY=y<br>
# CONFIG_F2FS_CHECK_FS is not set<br>
CONFIG_F2FS_FS_ENCRYPTION=y<br>
# CONFIG_F2FS_IO_TRACE is not set<br>
# CONFIG_F2FS_FAULT_INJECTION is not set<br>
# CONFIG_EFIVAR_FS is not set<br>
CONFIG_NETWORK_FILESYSTEMS=y<br>
# CONFIG_NFS_FS is not set<br>
# CONFIG_NFSD is not set<br>
# CONFIG_CEPH_FS is not set<br>
CONFIG_CIFS=y<br>
# CONFIG_CIFS_STATS is not set<br>
# CONFIG_CIFS_WEAK_PW_HASH is not set<br>
# CONFIG_CIFS_UPCALL is not set<br>
# CONFIG_CIFS_XATTR is not set<br>
CONFIG_CIFS_DEBUG=y<br>
# CONFIG_CIFS_DEBUG2 is not set<br>
# CONFIG_CIFS_DFS_UPCALL is not set<br>
# CONFIG_CIFS_SMB2 is not set<br>
# CONFIG_NCP_FS is not set<br>
# CONFIG_CODA_FS is not set<br>
# CONFIG_AFS_FS is not set<br>
CONFIG_NLS=y<br>
CONFIG_NLS_DEFAULT="iso8859-1"<br>
CONFIG_NLS_CODEPAGE_437=y<br>
# CONFIG_NLS_CODEPAGE_737 is not set<br>
# CONFIG_NLS_CODEPAGE_775 is not set<br>
# CONFIG_NLS_CODEPAGE_850 is not set<br>
# CONFIG_NLS_CODEPAGE_852 is not set<br>
# CONFIG_NLS_CODEPAGE_855 is not set<br>
# CONFIG_NLS_CODEPAGE_857 is not set<br>
# CONFIG_NLS_CODEPAGE_860 is not set<br>
# CONFIG_NLS_CODEPAGE_861 is not set<br>
# CONFIG_NLS_CODEPAGE_862 is not set<br>
# CONFIG_NLS_CODEPAGE_863 is not set<br>
# CONFIG_NLS_CODEPAGE_864 is not set<br>
# CONFIG_NLS_CODEPAGE_865 is not set<br>
# CONFIG_NLS_CODEPAGE_866 is not set<br>
# CONFIG_NLS_CODEPAGE_869 is not set<br>
CONFIG_NLS_CODEPAGE_936=y<br>
CONFIG_NLS_CODEPAGE_950=y<br>
# CONFIG_NLS_CODEPAGE_932 is not set<br>
# CONFIG_NLS_CODEPAGE_949 is not set<br>
# CONFIG_NLS_CODEPAGE_874 is not set<br>
# CONFIG_NLS_ISO8859_8 is not set<br>
# CONFIG_NLS_CODEPAGE_1250 is not set<br>
# CONFIG_NLS_CODEPAGE_1251 is not set<br>
CONFIG_NLS_ASCII=y<br>
CONFIG_NLS_ISO8859_1=y<br>
# CONFIG_NLS_ISO8859_2 is not set<br>
# CONFIG_NLS_ISO8859_3 is not set<br>
# CONFIG_NLS_ISO8859_4 is not set<br>
# CONFIG_NLS_ISO8859_5 is not set<br>
# CONFIG_NLS_ISO8859_6 is not set<br>
# CONFIG_NLS_ISO8859_7 is not set<br>
# CONFIG_NLS_ISO8859_9 is not set<br>
# CONFIG_NLS_ISO8859_13 is not set<br>
# CONFIG_NLS_ISO8859_14 is not set<br>
# CONFIG_NLS_ISO8859_15 is not set<br>
# CONFIG_NLS_KOI8_R is not set<br>
# CONFIG_NLS_KOI8_U is not set<br>
# CONFIG_NLS_MAC_ROMAN is not set<br>
# CONFIG_NLS_MAC_CELTIC is not set<br>
# CONFIG_NLS_MAC_CENTEURO is not set<br>
# CONFIG_NLS_MAC_CROATIAN is not set<br>
# CONFIG_NLS_MAC_CYRILLIC is not set<br>
# CONFIG_NLS_MAC_GAELIC is not set<br>
# CONFIG_NLS_MAC_GREEK is not set<br>
# CONFIG_NLS_MAC_ICELAND is not set<br>
# CONFIG_NLS_MAC_INUIT is not set<br>
# CONFIG_NLS_MAC_ROMANIAN is not set<br>
# CONFIG_NLS_MAC_TURKISH is not set<br>
CONFIG_NLS_UTF8=y<br>
# CONFIG_DLM is not set<br>
# CONFIG_FILE_TABLE_DEBUG is not set<br>
# CONFIG_VIRTUALIZATION is not set<br>
<br>
#<br>
# Kernel hacking<br>
#<br>
<br>
#<br>
# printk and dmesg options<br>
#<br>
CONFIG_PRINTK_TIME=y<br>
CONFIG_MESSAGE_LOGLEVEL_DEFAULT=4<br>
# CONFIG_LOG_BUF_MAGIC is not set<br>
# CONFIG_BOOT_PRINTK_DELAY is not set<br>
# CONFIG_DYNAMIC_DEBUG is not set<br>
<br>
#<br>
# Compile-time checks and compiler options<br>
#<br>
CONFIG_DEBUG_INFO=y<br>
# CONFIG_DEBUG_INFO_REDUCED is not set<br>
# CONFIG_DEBUG_INFO_SPLIT is not set<br>
# CONFIG_DEBUG_INFO_DWARF4 is not set<br>
CONFIG_ENABLE_WARN_DEPRECATED=y<br>
CONFIG_ENABLE_MUST_CHECK=y<br>
CONFIG_FRAME_WARN=2048<br>
# CONFIG_STRIP_ASM_SYMS is not set<br>
# CONFIG_READABLE_ASM is not set<br>
# CONFIG_UNUSED_SYMBOLS is not set<br>
# CONFIG_PAGE_OWNER is not set<br>
CONFIG_DEBUG_FS=y<br>
# CONFIG_HEADERS_CHECK is not set<br>
# CONFIG_DEBUG_SECTION_MISMATCH is not set<br>
CONFIG_ARCH_WANT_FRAME_POINTERS=y<br>
CONFIG_FRAME_POINTER=y<br>
# CONFIG_DEBUG_FORCE_WEAK_PER_CPU is not set<br>
CONFIG_MAGIC_SYSRQ=y<br>
CONFIG_MAGIC_SYSRQ_DEFAULT_ENABLE=0x1<br>
CONFIG_DEBUG_KERNEL=y<br>
<br>
#<br>
# Memory Debugging<br>
#<br>
# CONFIG_DEBUG_PAGEALLOC is not set<br>
# CONFIG_DEBUG_OBJECTS is not set<br>
# CONFIG_SLUB_STATS is not set<br>
CONFIG_HAVE_DEBUG_KMEMLEAK=y<br>
# CONFIG_DEBUG_KMEMLEAK is not set<br>
# CONFIG_DEBUG_STACK_USAGE is not set<br>
# CONFIG_DEBUG_VM is not set<br>
# CONFIG_DEBUG_MEMORY_INIT is not set<br>
# CONFIG_DEBUG_PER_CPU_MAPS is not set<br>
CONFIG_HAVE_ARCH_KASAN=y<br>
# CONFIG_DEBUG_SHIRQ is not set<br>
<br>
#<br>
# Debug Lockups and Hangs<br>
#<br>
# CONFIG_LOCKUP_DETECTOR is not set<br>
# CONFIG_DETECT_HUNG_TASK is not set<br>
# CONFIG_PANIC_ON_OOPS is not set<br>
CONFIG_PANIC_ON_OOPS_VALUE=0<br>
CONFIG_PANIC_TIMEOUT=5<br>
CONFIG_PANIC_ON_RECURSIVE_FAULT=y<br>
CONFIG_SCHED_DEBUG=y<br>
# CONFIG_PANIC_ON_SCHED_BUG is not set<br>
# CONFIG_PANIC_ON_RT_THROTTLING is not set<br>
CONFIG_SYSRQ_SCHED_DEBUG=y<br>
CONFIG_SCHEDSTATS=y<br>
# CONFIG_SCHED_STACK_END_CHECK is not set<br>
CONFIG_DEBUG_PREEMPT=y<br>
<br>
#<br>
# Lock Debugging (spinlocks, mutexes, etc...)<br>
#<br>
# CONFIG_DEBUG_RT_MUTEXES is not set<br>
# CONFIG_DEBUG_SPINLOCK is not set<br>
# CONFIG_DEBUG_MUTEXES is not set<br>
# CONFIG_DEBUG_WW_MUTEX_SLOWPATH is not set<br>
# CONFIG_DEBUG_LOCK_ALLOC is not set<br>
# CONFIG_PROVE_LOCKING is not set<br>
# CONFIG_LOCK_STAT is not set<br>
# CONFIG_DEBUG_ATOMIC_SLEEP is not set<br>
# CONFIG_DEBUG_LOCKING_API_SELFTESTS is not set<br>
# CONFIG_LOCK_TORTURE_TEST is not set<br>
CONFIG_STACKTRACE=y<br>
# CONFIG_DEBUG_KOBJECT is not set<br>
CONFIG_HAVE_DEBUG_BUGVERBOSE=y<br>
CONFIG_DEBUG_BUGVERBOSE=y<br>
# CONFIG_DEBUG_LIST is not set<br>
# CONFIG_DEBUG_PI_LIST is not set<br>
# CONFIG_DEBUG_SG is not set<br>
# CONFIG_DEBUG_NOTIFIERS is not set<br>
# CONFIG_DEBUG_CREDENTIALS is not set<br>
<br>
#<br>
# RCU Debugging<br>
#<br>
# CONFIG_SPARSE_RCU_POINTER is not set<br>
# CONFIG_TORTURE_TEST is not set<br>
# CONFIG_RCU_TORTURE_TEST is not set<br>
CONFIG_RCU_CPU_STALL_TIMEOUT=21<br>
CONFIG_RCU_CPU_STALL_VERBOSE=y<br>
# CONFIG_RCU_CPU_STALL_INFO is not set<br>
# CONFIG_RCU_TRACE is not set<br>
# CONFIG_DEBUG_BLOCK_EXT_DEVT is not set<br>
# CONFIG_NOTIFIER_ERROR_INJECTION is not set<br>
# CONFIG_FAULT_INJECTION is not set<br>
CONFIG_NOP_TRACER=y<br>
CONFIG_HAVE_FUNCTION_TRACER=y<br>
CONFIG_HAVE_FUNCTION_GRAPH_TRACER=y<br>
CONFIG_HAVE_DYNAMIC_FTRACE=y<br>
CONFIG_HAVE_FTRACE_MCOUNT_RECORD=y<br>
CONFIG_HAVE_SYSCALL_TRACEPOINTS=y<br>
CONFIG_HAVE_C_RECORDMCOUNT=y<br>
CONFIG_TRACE_CLOCK=y<br>
CONFIG_RING_BUFFER=y<br>
CONFIG_EVENT_TRACING=y<br>
CONFIG_CONTEXT_SWITCH_TRACER=y<br>
# CONFIG_MSM_RTB is not set<br>
CONFIG_IPC_LOGGING=y<br>
CONFIG_TRACING=y<br>
CONFIG_GENERIC_TRACER=y<br>
CONFIG_TRACING_SUPPORT=y<br>
CONFIG_FTRACE=y<br>
CONFIG_FUNCTION_TRACER=y<br>
CONFIG_FUNCTION_GRAPH_TRACER=y<br>
# CONFIG_IRQSOFF_TRACER is not set<br>
# CONFIG_PREEMPT_TRACER is not set<br>
# CONFIG_SCHED_TRACER is not set<br>
# CONFIG_FTRACE_SYSCALLS is not set<br>
# CONFIG_TRACER_SNAPSHOT is not set<br>
CONFIG_BRANCH_PROFILE_NONE=y<br>
# CONFIG_PROFILE_ANNOTATED_BRANCHES is not set<br>
# CONFIG_PROFILE_ALL_BRANCHES is not set<br>
# CONFIG_STACK_TRACER is not set<br>
# CONFIG_BLK_DEV_IO_TRACE is not set<br>
# CONFIG_PROBE_EVENTS is not set<br>
CONFIG_DYNAMIC_FTRACE=y<br>
# CONFIG_FUNCTION_PROFILER is not set<br>
CONFIG_CPU_FREQ_SWITCH_PROFILER=y<br>
CONFIG_FTRACE_MCOUNT_RECORD=y<br>
# CONFIG_FTRACE_STARTUP_TEST is not set<br>
# CONFIG_TRACEPOINT_BENCHMARK is not set<br>
# CONFIG_RING_BUFFER_BENCHMARK is not set<br>
# CONFIG_RING_BUFFER_STARTUP_TEST is not set<br>
<br>
#<br>
# Runtime Testing<br>
#<br>
# CONFIG_LKDTM is not set<br>
# CONFIG_TEST_LIST_SORT is not set<br>
# CONFIG_BACKTRACE_SELF_TEST is not set<br>
# CONFIG_RBTREE_TEST is not set<br>
# CONFIG_INTERVAL_TREE_TEST is not set<br>
# CONFIG_PERCPU_TEST is not set<br>
# CONFIG_ATOMIC64_SELFTEST is not set<br>
# CONFIG_TEST_STRING_HELPERS is not set<br>
# CONFIG_TEST_KSTRTOX is not set<br>
# CONFIG_TEST_RHASHTABLE is not set<br>
# CONFIG_DMA_API_DEBUG is not set<br>
# CONFIG_TEST_LKM is not set<br>
# CONFIG_TEST_USER_COPY is not set<br>
# CONFIG_TEST_BPF is not set<br>
# CONFIG_TEST_FIRMWARE is not set<br>
# CONFIG_TEST_UDELAY is not set<br>
# CONFIG_PANIC_ON_DATA_CORRUPTION is not set<br>
# CONFIG_MEMTEST is not set<br>
# CONFIG_SAMPLES is not set<br>
CONFIG_HAVE_ARCH_KGDB=y<br>
# CONFIG_KGDB is not set<br>
CONFIG_ARCH_HAS_UBSAN_SANITIZE_ALL=y<br>
# CONFIG_UBSAN is not set<br>
# CONFIG_ARM64_PTDUMP is not set<br>
# CONFIG_STRICT_DEVMEM is not set<br>
# CONFIG_PID_IN_CONTEXTIDR is not set<br>
# CONFIG_ARM64_RANDOMIZE_TEXT_OFFSET is not set<br>
# CONFIG_DEBUG_SET_MODULE_RONX is not set<br>
# CONFIG_FREE_PAGES_RDONLY is not set<br>
CONFIG_DEBUG_RODATA=y<br>
# CONFIG_DEBUG_ALIGN_RODATA is not set<br>
# CONFIG_CORESIGHT_LINKS_AND_SINKS is not set<br>
# CONFIG_CORESIGHT_SOURCE_ETM4X is not set<br>
<br>
#<br>
# Security options<br>
#<br>
CONFIG_KEYS=y<br>
CONFIG_KEYS_COMPAT=y<br>
# CONFIG_PERSISTENT_KEYRINGS is not set<br>
# CONFIG_BIG_KEYS is not set<br>
CONFIG_ENCRYPTED_KEYS=y<br>
# CONFIG_KEYS_DEBUG_PROC_KEYS is not set<br>
<br>
#<br>
# Qualcomm Technologies, Inc Per File Encryption security device drivers<br>
#<br>
# CONFIG_PFK is not set<br>
# CONFIG_SECURITY_DMESG_RESTRICT is not set<br>
# CONFIG_SECURITY_PERF_EVENTS_RESTRICT is not set<br>
CONFIG_SECURITY=y<br>
# CONFIG_SECURITYFS is not set<br>
CONFIG_SECURITY_NETWORK=y<br>
# CONFIG_SECURITY_NETWORK_XFRM is not set<br>
# CONFIG_SECURITY_PATH is not set<br>
CONFIG_LSM_MMAP_MIN_ADDR=4096<br>
CONFIG_SECURITY_SELINUX=y<br>
CONFIG_SECURITY_SELINUX_BOOTPARAM=y<br>
CONFIG_SECURITY_SELINUX_BOOTPARAM_VALUE=1<br>
# CONFIG_SECURITY_SELINUX_DISABLE is not set<br>
CONFIG_SECURITY_SELINUX_DEVELOP=y<br>
CONFIG_SECURITY_SELINUX_AVC_STATS=y<br>
CONFIG_SECURITY_SELINUX_CHECKREQPROT_VALUE=1<br>
# CONFIG_SECURITY_SELINUX_POLICYDB_VERSION_MAX is not set<br>
# CONFIG_SECURITY_SMACK is not set<br>
# CONFIG_SECURITY_TOMOYO is not set<br>
# CONFIG_SECURITY_APPARMOR is not set<br>
# CONFIG_SECURITY_YAMA is not set<br>
CONFIG_INTEGRITY=y<br>
# CONFIG_INTEGRITY_SIGNATURE is not set<br>
CONFIG_INTEGRITY_AUDIT=y<br>
# CONFIG_IMA is not set<br>
# CONFIG_EVM is not set<br>
CONFIG_DEFAULT_SECURITY_SELINUX=y<br>
# CONFIG_DEFAULT_SECURITY_DAC is not set<br>
CONFIG_DEFAULT_SECURITY="selinux"<br>
CONFIG_CRYPTO=y<br>
<br>
#<br>
# Crypto core or helper<br>
#<br>
CONFIG_CRYPTO_ALGAPI=y<br>
CONFIG_CRYPTO_ALGAPI2=y<br>
CONFIG_CRYPTO_AEAD=y<br>
CONFIG_CRYPTO_AEAD2=y<br>
CONFIG_CRYPTO_BLKCIPHER=y<br>
CONFIG_CRYPTO_BLKCIPHER2=y<br>
CONFIG_CRYPTO_HASH=y<br>
CONFIG_CRYPTO_HASH2=y<br>
CONFIG_CRYPTO_RNG=y<br>
CONFIG_CRYPTO_RNG2=y<br>
CONFIG_CRYPTO_PCOMP2=y<br>
CONFIG_CRYPTO_MANAGER=y<br>
CONFIG_CRYPTO_MANAGER2=y<br>
# CONFIG_CRYPTO_USER is not set<br>
CONFIG_CRYPTO_MANAGER_DISABLE_TESTS=y<br>
CONFIG_CRYPTO_GF128MUL=y<br>
CONFIG_CRYPTO_NULL=y<br>
# CONFIG_CRYPTO_PCRYPT is not set<br>
CONFIG_CRYPTO_WORKQUEUE=y<br>
CONFIG_CRYPTO_CRYPTD=y<br>
# CONFIG_CRYPTO_MCRYPTD is not set<br>
CONFIG_CRYPTO_AUTHENC=y<br>
# CONFIG_CRYPTO_TEST is not set<br>
CONFIG_CRYPTO_ABLK_HELPER=y<br>
<br>
#<br>
# Authenticated Encryption with Associated Data<br>
#<br>
# CONFIG_CRYPTO_CCM is not set<br>
# CONFIG_CRYPTO_GCM is not set<br>
CONFIG_CRYPTO_SEQIV=y<br>
<br>
#<br>
# Block modes<br>
#<br>
CONFIG_CRYPTO_CBC=y<br>
CONFIG_CRYPTO_CTR=y<br>
CONFIG_CRYPTO_CTS=y<br>
CONFIG_CRYPTO_ECB=y<br>
# CONFIG_CRYPTO_LRW is not set<br>
# CONFIG_CRYPTO_PCBC is not set<br>
CONFIG_CRYPTO_XTS=y<br>
<br>
#<br>
# Hash modes<br>
#<br>
CONFIG_CRYPTO_CMAC=y<br>
CONFIG_CRYPTO_HMAC=y<br>
CONFIG_CRYPTO_XCBC=y<br>
# CONFIG_CRYPTO_VMAC is not set<br>
<br>
#<br>
# Digest<br>
#<br>
CONFIG_CRYPTO_CRC32C=y<br>
CONFIG_CRYPTO_CRC32=y<br>
# CONFIG_CRYPTO_CRCT10DIF is not set<br>
# CONFIG_CRYPTO_GHASH is not set<br>
CONFIG_CRYPTO_MD4=y<br>
CONFIG_CRYPTO_MD5=y<br>
# CONFIG_CRYPTO_MICHAEL_MIC is not set<br>
# CONFIG_CRYPTO_RMD128 is not set<br>
# CONFIG_CRYPTO_RMD160 is not set<br>
# CONFIG_CRYPTO_RMD256 is not set<br>
# CONFIG_CRYPTO_RMD320 is not set<br>
CONFIG_CRYPTO_SHA1=y<br>
CONFIG_CRYPTO_SHA256=y<br>
CONFIG_CRYPTO_SHA512=y<br>
# CONFIG_CRYPTO_TGR192 is not set<br>
# CONFIG_CRYPTO_WP512 is not set<br>
<br>
#<br>
# Ciphers<br>
#<br>
CONFIG_CRYPTO_AES=y<br>
# CONFIG_CRYPTO_ANUBIS is not set<br>
CONFIG_CRYPTO_ARC4=y<br>
# CONFIG_CRYPTO_BLOWFISH is not set<br>
# CONFIG_CRYPTO_CAMELLIA is not set<br>
# CONFIG_CRYPTO_CAST5 is not set<br>
# CONFIG_CRYPTO_CAST6 is not set<br>
CONFIG_CRYPTO_DES=y<br>
# CONFIG_CRYPTO_FCRYPT is not set<br>
# CONFIG_CRYPTO_KHAZAD is not set<br>
# CONFIG_CRYPTO_SALSA20 is not set<br>
# CONFIG_CRYPTO_SEED is not set<br>
# CONFIG_CRYPTO_SERPENT is not set<br>
# CONFIG_CRYPTO_SPECK is not set<br>
# CONFIG_CRYPTO_TEA is not set<br>
CONFIG_CRYPTO_TWOFISH=y<br>
CONFIG_CRYPTO_TWOFISH_COMMON=y<br>
<br>
#<br>
# Compression<br>
#<br>
CONFIG_CRYPTO_DEFLATE=y<br>
# CONFIG_CRYPTO_ZLIB is not set<br>
CONFIG_CRYPTO_LZO=y<br>
# CONFIG_CRYPTO_LZ4 is not set<br>
# CONFIG_CRYPTO_LZ4HC is not set<br>
<br>
#<br>
# Random Number Generation<br>
#<br>
CONFIG_CRYPTO_ANSI_CPRNG=y<br>
# CONFIG_CRYPTO_DRBG_MENU is not set<br>
# CONFIG_CRYPTO_USER_API_HASH is not set<br>
# CONFIG_CRYPTO_USER_API_SKCIPHER is not set<br>
CONFIG_CRYPTO_HASH_INFO=y<br>
CONFIG_CRYPTO_HW=y<br>
CONFIG_CRYPTO_DEV_QCE50=y<br>
# CONFIG_FIPS_ENABLE is not set<br>
CONFIG_CRYPTO_DEV_QCRYPTO=y<br>
CONFIG_CRYPTO_DEV_QCOM_MSM_QCE=y<br>
CONFIG_CRYPTO_DEV_QCEDEV=y<br>
CONFIG_CRYPTO_DEV_OTA_CRYPTO=y<br>
CONFIG_CRYPTO_DEV_QCOM_ICE=y<br>
# CONFIG_CRYPTO_DEV_CCP is not set<br>
CONFIG_ASYMMETRIC_KEY_TYPE=y<br>
CONFIG_ASYMMETRIC_PUBLIC_KEY_SUBTYPE=y<br>
CONFIG_PUBLIC_KEY_ALGO_RSA=y<br>
CONFIG_X509_CERTIFICATE_PARSER=y<br>
# CONFIG_PKCS7_MESSAGE_PARSER is not set<br>
CONFIG_ARM64_CRYPTO=y<br>
CONFIG_CRYPTO_SHA1_ARM64_CE=y<br>
CONFIG_CRYPTO_SHA2_ARM64_CE=y<br>
CONFIG_CRYPTO_GHASH_ARM64_CE=y<br>
CONFIG_CRYPTO_AES_ARM64_CE=y<br>
CONFIG_CRYPTO_AES_ARM64_CE_CCM=y<br>
CONFIG_CRYPTO_AES_ARM64_CE_BLK=y<br>
CONFIG_CRYPTO_AES_ARM64_NEON_BLK=y<br>
CONFIG_CRYPTO_CRC32_ARM64=y<br>
CONFIG_BINARY_PRINTF=y<br>
<br>
#<br>
# Library routines<br>
#<br>
CONFIG_BITREVERSE=y<br>
CONFIG_GENERIC_STRNCPY_FROM_USER=y<br>
CONFIG_GENERIC_STRNLEN_USER=y<br>
CONFIG_GENERIC_NET_UTILS=y<br>
CONFIG_GENERIC_PCI_IOMAP=y<br>
CONFIG_GENERIC_IOMAP=y<br>
CONFIG_GENERIC_IO=y<br>
CONFIG_ARCH_USE_CMPXCHG_LOCKREF=y<br>
CONFIG_CRC_CCITT=y<br>
CONFIG_CRC16=y<br>
# CONFIG_CRC_T10DIF is not set<br>
# CONFIG_CRC_ITU_T is not set<br>
CONFIG_CRC32=y<br>
# CONFIG_CRC32_SELFTEST is not set<br>
CONFIG_CRC32_SLICEBY8=y<br>
# CONFIG_CRC32_SLICEBY4 is not set<br>
# CONFIG_CRC32_SARWATE is not set<br>
# CONFIG_CRC32_BIT is not set<br>
# CONFIG_CRC7 is not set<br>
CONFIG_LIBCRC32C=y<br>
# CONFIG_CRC8 is not set<br>
CONFIG_AUDIT_GENERIC=y<br>
CONFIG_AUDIT_ARCH_COMPAT_GENERIC=y<br>
CONFIG_AUDIT_COMPAT_GENERIC=y<br>
# CONFIG_RANDOM32_SELFTEST is not set<br>
CONFIG_ZLIB_INFLATE=y<br>
CONFIG_ZLIB_DEFLATE=y<br>
CONFIG_LZO_COMPRESS=y<br>
CONFIG_LZO_DECOMPRESS=y<br>
CONFIG_LZ4_COMPRESS=y<br>
CONFIG_LZ4_DECOMPRESS=y<br>
CONFIG_XZ_DEC=y<br>
CONFIG_XZ_DEC_X86=y<br>
CONFIG_XZ_DEC_POWERPC=y<br>
CONFIG_XZ_DEC_IA64=y<br>
CONFIG_XZ_DEC_ARM=y<br>
CONFIG_XZ_DEC_ARMTHUMB=y<br>
CONFIG_XZ_DEC_SPARC=y<br>
CONFIG_XZ_DEC_BCJ=y<br>
# CONFIG_XZ_DEC_TEST is not set<br>
CONFIG_DECOMPRESS_GZIP=y<br>
CONFIG_DECOMPRESS_BZIP2=y<br>
CONFIG_DECOMPRESS_LZMA=y<br>
CONFIG_GENERIC_ALLOCATOR=y<br>
CONFIG_REED_SOLOMON=y<br>
CONFIG_REED_SOLOMON_ENC8=y<br>
CONFIG_REED_SOLOMON_DEC8=y<br>
CONFIG_TEXTSEARCH=y<br>
CONFIG_TEXTSEARCH_KMP=y<br>
CONFIG_TEXTSEARCH_BM=y<br>
CONFIG_TEXTSEARCH_FSM=y<br>
CONFIG_ASSOCIATIVE_ARRAY=y<br>
CONFIG_HAS_IOMEM=y<br>
CONFIG_HAS_IOPORT_MAP=y<br>
CONFIG_HAS_DMA=y<br>
CONFIG_CPU_RMAP=y<br>
CONFIG_DQL=y<br>
CONFIG_NLATTR=y<br>
CONFIG_ARCH_HAS_ATOMIC64_DEC_IF_POSITIVE=y<br>
# CONFIG_AVERAGE is not set<br>
CONFIG_CLZ_TAB=y<br>
# CONFIG_CORDIC is not set<br>
# CONFIG_DDR is not set<br>
CONFIG_MPILIB=y<br>
CONFIG_LIBFDT=y<br>
CONFIG_OID_REGISTRY=y<br>
CONFIG_UCS2_STRING=y<br>
CONFIG_ARCH_HAS_SG_CHAIN=y<br>
CONFIG_QMI_ENCDEC=y<br>
# CONFIG_QMI_ENCDEC_DEBUG is not set<br>
# CONFIG_STRICT_MEMORY_RWX is not set<br>
</details><br>


Исходные коды данного ядра найдены в https://github.com/lordarcadius/electrablue_mido_nougat<br>
Возьмем его
```console
HABUILD_SDK [mido]:~/hadk$ mv kernel/xiaomi/msm8953/ kernel/xiaomi/msm8953_orig
HABUILD_SDK [mido]:~/hadk$ git clone https://github.com/lordarcadius/electrablue_mido_nougat.git kernel/xiaomi/msm8953
```
<details>
Cloning into 'kernel/xiaomi/msm8953'...<br>
remote: Enumerating objects: 4122385, done.<br>
remote: Total 4122385 (delta 0), reused 0 (delta 0), pack-reused 4122385<br>
Receiving objects: 100% (4122385/4122385), 742.81 MiB | 8.33 MiB/s, done.<br>
Resolving deltas: 100% (3455693/3455693), done.<br>
Checking out files: 100% (51997/51997), done.<br>
</details><br>

Скопируем со смартфона рабочую конфигурацию ядра по нужному адресу для последующей сборки
```console
sh-3.2# scp mido_sf_defconfig stalker@192.168.2.1:/home/stalker/hadk/kernel/xiaomi/msm8953/arch/arm64/configs
```
<details>
stalker@192.168.2.1's password:<br>
mido_sf_defconfig                                                                              100%  122KB  14.6MB/s   00:00<br>
</details><br>

Скомпилируем ядро
```console
HABUILD_SDK [mido]:~/hadk$ source build/envsetup.sh
```
<details>
including vendor/cm/vendorsetup.sh<br>
including vendor/cm/bash_completion/git.bash<br>
including vendor/cm/bash_completion/repo.bash<br>
</details><br>

```console
HABUILD_SDK [mido]:~/hadk$ export USE_CCACHE=1
HABUILD_SDK [mido]:~/hadk$ breakfast $DEVICE
```
<details>
including vendor/cm/vendorsetup.sh<br>
Trying dependencies-only mode on a non-existing device tree?<br>
<br>
============================================<br>
PLATFORM_VERSION_CODENAME=REL<br>
PLATFORM_VERSION=7.1.2<br>
LINEAGE_VERSION=14.1-20190710-UNOFFICIAL-mido<br>
TARGET_PRODUCT=lineage_mido<br>
TARGET_BUILD_VARIANT=userdebug<br>
TARGET_BUILD_TYPE=release<br>
TARGET_BUILD_APPS=<br>
TARGET_ARCH=arm64<br>
TARGET_ARCH_VARIANT=armv8-a<br>
TARGET_CPU_VARIANT=generic<br>
TARGET_2ND_ARCH=arm<br>
TARGET_2ND_ARCH_VARIANT=armv7-a-neon<br>
TARGET_2ND_CPU_VARIANT=cortex-a53<br>
HOST_ARCH=x86_64<br>
HOST_2ND_ARCH=x86<br>
HOST_OS=linux<br>
HOST_OS_EXTRA=Linux-4.15.0-54-generic-x86_64-with-Ubuntu-14.04-trusty<br>
HOST_CROSS_OS=windows<br>
HOST_CROSS_ARCH=x86<br>
HOST_CROSS_2ND_ARCH=x86_64<br>
HOST_BUILD_TYPE=release<br>
BUILD_ID=NJH47F<br>
OUT_DIR=/home/stalker/hadk/out<br>
============================================<br>
</details><br>

```console
HABUILD_SDK [mido]:~/hadk$ make -j$(nproc --all) hybris-hal
```
<details>
============================================<br>
PLATFORM_VERSION_CODENAME=REL<br>
PLATFORM_VERSION=7.1.2<br>
LINEAGE_VERSION=14.1-20190710-UNOFFICIAL-mido<br>
TARGET_PRODUCT=lineage_mido<br>
TARGET_BUILD_VARIANT=userdebug<br>
TARGET_BUILD_TYPE=release<br>
TARGET_BUILD_APPS=<br>
TARGET_ARCH=arm64<br>
TARGET_ARCH_VARIANT=armv8-a<br>
TARGET_CPU_VARIANT=generic<br>
TARGET_2ND_ARCH=arm<br>
TARGET_2ND_ARCH_VARIANT=armv7-a-neon<br>
TARGET_2ND_CPU_VARIANT=cortex-a53<br>
HOST_ARCH=x86_64<br>
HOST_2ND_ARCH=x86<br>
HOST_OS=linux<br>
HOST_OS_EXTRA=Linux-4.15.0-54-generic-x86_64-with-Ubuntu-14.04-trusty<br>
HOST_CROSS_OS=windows<br>
HOST_CROSS_ARCH=x86<br>
HOST_CROSS_2ND_ARCH=x86_64<br>
HOST_BUILD_TYPE=release<br>
BUILD_ID=NJH47F<br>
OUT_DIR=/home/stalker/hadk/out<br>
============================================<br>
Running kati to generate build-lineage_mido.ninja...<br>
Environment variable BUILD_NUMBER was modified (4a0ab6dddd = 1e6529fe31), regenerating...<br>
============================================<br>
PLATFORM_VERSION_CODENAME=REL<br>
PLATFORM_VERSION=7.1.2<br>
LINEAGE_VERSION=14.1-20190710-UNOFFICIAL-mido<br>
TARGET_PRODUCT=lineage_mido<br>
TARGET_BUILD_VARIANT=userdebug<br>
TARGET_BUILD_TYPE=release<br>
TARGET_BUILD_APPS=<br>
TARGET_ARCH=arm64<br>
TARGET_ARCH_VARIANT=armv8-a<br>
TARGET_CPU_VARIANT=generic<br>
TARGET_2ND_ARCH=arm<br>
TARGET_2ND_ARCH_VARIANT=armv7-a-neon<br>
TARGET_2ND_CPU_VARIANT=cortex-a53<br>
HOST_ARCH=x86_64<br>
HOST_2ND_ARCH=x86<br>
HOST_OS=linux<br>
HOST_OS_EXTRA=Linux-4.15.0-54-generic-x86_64-with-Ubuntu-14.04-trusty<br>
HOST_CROSS_OS=windows<br>
HOST_CROSS_ARCH=x86<br>
HOST_CROSS_2ND_ARCH=x86_64<br>
HOST_BUILD_TYPE=release<br>
BUILD_ID=NJH47F<br>
OUT_DIR=/home/stalker/hadk/out<br>
============================================<br>
build/core/binary.mk:37: hal3-test-app uses kernel headers, but does not depend on them!<br>
external/speex/Android.mk:56: TODOArm64: enable neon in libspeex<br>
perl: warning: Setting locale failed.<br>
perl: warning: Please check that your locale settings:<br>
	LANGUAGE = (unset),<br>
	LC_ALL = (unset),<br>
	LC_MESSAGES = "en_US.UTF-8",<br>
	LC_CTYPE = "en_US.UTF-8",<br>
	LANG = (unset)<br>
    are supported and installed on your system.<br>
perl: warning: Falling back to the standard locale ("C").<br>
perl: warning: Setting locale failed.<br>
perl: warning: Please check that your locale settings:<br>
	LANGUAGE = (unset),<br>
	LC_ALL = (unset),<br>
	LC_MESSAGES = "en_US.UTF-8",<br>
	LC_CTYPE = "en_US.UTF-8",<br>
	LANG = (unset)<br>
    are supported and installed on your system.<br>
perl: warning: Falling back to the standard locale ("C").<br>
hybris/hybris-boot/Android.mk:71: ********************* /boot appears to live on /dev/block/bootdevice/by-name/boot<br>
hybris/hybris-boot/Android.mk:72: ********************* /data appears to live on /dev/block/bootdevice/by-name/userdata<br>
build/core/Makefile:34: warning: overriding commands for target `/home/stalker/hadk/out/target/product/mido/system/bin/wcnss_service'<br>
build/core/base_rules.mk:320: warning: ignoring old commands for target `/home/stalker/hadk/out/target/product/mido/system/bin/wcnss_service'<br>
Starting build with ninja<br>
ninja: Entering directory `.'<br>
[ 29% 5/17] build /home/stalker/hadk/out/target/product/mido/obj/ROOT/hybris-boot_intermediates/init<br>
Fixing mount-points for device mido<br>
[ 35% 6/17] build /home/stalker/hadk/out/target/product/mido/obj/ROOT/hybris-recovery_intermediates/init<br>
Fixing mount-points for device mido<br>
[ 41% 7/17] Target buildinfo: /home/stalker/hadk/out/target/product/mido/obj/ETC/system_build_prop_intermediates/build.prop<br>
Target buildinfo from: device/xiaomi/mido/system.prop<br>
[ 52% 9/17] Making initramfs : /home/stalker/hadk/out/target/product/mido/obj/ROOT/hybris-recovery_intermediates/recovery-initramfs.gz<br>
2953 blocks<br>
[ 52% 9/17] Making initramfs : /home/stalker/hadk/out/target/product/mido/obj/ROOT/hybris-boot_intermediates/boot-initramfs.gz<br>
2953 blocks<br>
[ 52% 9/17] Building Kernel Config<br>
make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  GEN     ./Makefile<br>
  HOSTCC  scripts/basic/fixdep<br>
  HOSTCC  scripts/basic/bin2c<br>
  HOSTCC  scripts/kconfig/conf.o<br>
  SHIPPED scripts/kconfig/zconf.tab.c<br>
  SHIPPED scripts/kconfig/zconf.lex.c<br>
  SHIPPED scripts/kconfig/zconf.hash.c<br>
  HOSTCC  scripts/kconfig/zconf.tab.o<br>
  HOSTLD  scripts/kconfig/conf<br>
warning: (SND_SOC_APQ8084 && SND_SOC_MSM8994 && SND_SOC_MSM8996 && SND_SOC_MSM8X16 && SND_SOC_MDM9607 && SND_SOC_MDM9640) selects SND_SOC_WCD9330 which has unmet direct dependencies (SOUND && !M68K && !UML && SND && SND_SOC && WCD9330_CODEC)<br>
warning: (SND_SOC_APQ8084 && SND_SOC_MSM8994 && SND_SOC_MSM8996 && SND_SOC_MSM8X16 && SND_SOC_MDM9607 && SND_SOC_MDM9640) selects SND_SOC_WCD9330 which has unmet direct dependencies (SOUND && !M68K && !UML && SND && SND_SOC && WCD9330_CODEC)<br>
#<br>
# configuration written to .config<br>
#<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  GEN     ./Makefile<br>
scripts/kconfig/conf --savedefconfig=defconfig Kconfig<br>
warning: (SND_SOC_APQ8084 && SND_SOC_MSM8994 && SND_SOC_MSM8996 && SND_SOC_MSM8X16 && SND_SOC_MDM9607 && SND_SOC_MDM9640) selects SND_SOC_WCD9330 which has unmet direct dependencies (SOUND && !M68K && !UML && SND && SND_SOC && WCD9330_CODEC)<br>
warning: (SND_SOC_APQ8084 && SND_SOC_MSM8994 && SND_SOC_MSM8996 && SND_SOC_MSM8X16 && SND_SOC_MDM9607 && SND_SOC_MDM9640) selects SND_SOC_WCD9330 which has unmet direct dependencies (SOUND && !M68K && !UML && SND && SND_SOC && WCD9330_CODEC)<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
[ 58% 10/17] Building Kernel Headers<br>
make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  GEN     ./Makefile<br>
warning: (SND_SOC_APQ8084 && SND_SOC_MSM8994 && SND_SOC_MSM8996 && SND_SOC_MSM8X16 && SND_SOC_MDM9607 && SND_SOC_MDM9640) selects SND_SOC_WCD9330 which has unmet direct dependencies (SOUND && !M68K && !UML && SND && SND_SOC && WCD9330_CODEC)<br>
warning: (SND_SOC_APQ8084 && SND_SOC_MSM8994 && SND_SOC_MSM8996 && SND_SOC_MSM8X16 && SND_SOC_MDM9607 && SND_SOC_MDM9640) selects SND_SOC_WCD9330 which has unmet direct dependencies (SOUND && !M68K && !UML && SND && SND_SOC && WCD9330_CODEC)<br>
#<br>
# configuration written to .config<br>
#<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  CHK     include/generated/uapi/linux/version.h<br>
  UPD     include/generated/uapi/linux/version.h<br>
  HOSTCC  scripts/unifdef<br>
  INSTALL usr/include/uapi/ (0 file)<br>
  INSTALL usr/include/mtd/ (5 files)<br>
  INSTALL usr/include/drm/ (18 files)<br>
  INSTALL usr/include/misc/ (1 file)<br>
  INSTALL usr/include/asm-generic/ (35 files)<br>
  INSTALL usr/include/rdma/ (6 files)<br>
  INSTALL usr/include/scsi/ (5 files)<br>
  INSTALL usr/include/media/ (21 files)<br>
  INSTALL usr/include/sound/ (19 files)<br>
  INSTALL usr/include/video/ (5 files)<br>
  INSTALL usr/include/xen/ (4 files)<br>
  INSTALL usr/include/scsi/fc/ (4 files)<br>
  INSTALL usr/include/scsi/ufs/ (2 files)<br>
  INSTALL usr/include/linux/../../../usr/include/linux/staging/android/uapi/ (2 files)<br>
  INSTALL usr/include/linux/byteorder/ (2 files)<br>
  INSTALL usr/include/linux/can/ (5 files)<br>
  INSTALL usr/include/linux/caif/ (2 files)<br>
  INSTALL usr/include/linux/dvb/ (8 files)<br>
  INSTALL usr/include/linux/hdlc/ (1 file)<br>
  INSTALL usr/include/linux/hsi/ (1 file)<br>
  INSTALL usr/include/linux/isdn/ (1 file)<br>
  INSTALL usr/include/linux/mmc/ (3 files)<br>
  INSTALL usr/include/linux/netfilter_arp/ (2 files)<br>
  INSTALL usr/include/linux/netfilter_bridge/ (17 files)<br>
  INSTALL usr/include/linux/netfilter/ (85 files)<br>
  INSTALL usr/include/linux/netfilter_ipv6/ (12 files)<br>
  INSTALL usr/include/linux/netfilter_ipv4/ (10 files)<br>
  INSTALL usr/include/linux/mfd/wcd9xxx/ (2 files)<br>
  INSTALL usr/include/linux/nfc/ (1 file)<br>
  INSTALL usr/include/linux/netfilter/ipset/ (4 files)<br>
  INSTALL usr/include/linux/nfsd/ (5 files)<br>
  INSTALL usr/include/linux/mfd/ (1 file)<br>
  INSTALL usr/include/linux/raid/ (2 files)<br>
  INSTALL usr/include/linux/spi/ (1 file)<br>
  INSTALL usr/include/linux/sunrpc/ (1 file)<br>
  INSTALL usr/include/linux/tc_act/ (8 files)<br>
  INSTALL usr/include/linux/tc_ematch/ (4 files)<br>
  REMOVE  lirc.h spcom.h<br>
  INSTALL usr/include/linux/usb/ (11 files)<br>
  INSTALL usr/include/linux/ (469 files)<br>
  INSTALL usr/include/linux/wimax/ (1 file)<br>
  INSTALL usr/include/asm/ (35 files)<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
[ 64% 11/17] Building Kernel<br>
make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  GEN     ./Makefile<br>
scripts/kconfig/conf --silentoldconfig Kconfig<br>
warning: (SND_SOC_APQ8084 && SND_SOC_MSM8994 && SND_SOC_MSM8996 && SND_SOC_MSM8X16 && SND_SOC_MDM9607 && SND_SOC_MDM9640) selects SND_SOC_WCD9330 which has unmet direct dependencies (SOUND && !M68K && !UML && SND && SND_SOC && WCD9330_CODEC)<br>
warning: (SND_SOC_APQ8084 && SND_SOC_MSM8994 && SND_SOC_MSM8996 && SND_SOC_MSM8X16 && SND_SOC_MDM9607 && SND_SOC_MDM9640) selects SND_SOC_WCD9330 which has unmet direct dependencies (SOUND && !M68K && !UML && SND && SND_SOC && WCD9330_CODEC)<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  CHK     include/config/kernel.release<br>
  UPD     include/config/kernel.release<br>
  GEN     ./Makefile<br>
  CHK     include/generated/uapi/linux/version.h<br>
  CHK     include/generated/utsrelease.h<br>
  UPD     include/generated/utsrelease.h<br>
  Using /home/stalker/hadk/kernel/xiaomi/msm8953 as source for kernel<br>
  HOSTCC  scripts/kallsyms<br>
  HOSTCC  scripts/conmakehash<br>
  HOSTCC  scripts/recordmcount<br>
  HOSTCC  scripts/sortextable<br>
  HOSTCC  scripts/asn1_compiler<br>
  HOSTCC  scripts/genksyms/genksyms.o<br>
  SHIPPED scripts/genksyms/parse.tab.c<br>
  SHIPPED scripts/genksyms/lex.lex.c<br>
  SHIPPED scripts/genksyms/keywords.hash.c<br>
  SHIPPED scripts/genksyms/parse.tab.h<br>
  HOSTCC  scripts/genksyms/parse.tab.o<br>
  HOSTCC  scripts/genksyms/lex.lex.o<br>
  CC      scripts/mod/empty.o<br>
  HOSTCC  scripts/dtc/dtc.o<br>
  HOSTCC  scripts/dtc/flattree.o<br>
  HOSTCC  scripts/mod/mk_elfconfig<br>
  HOSTCC  scripts/dtc/fstree.o<br>
  HOSTCC  scripts/selinux/genheaders/genheaders<br>
  HOSTCC  scripts/dtc/data.o<br>
  HOSTCC  scripts/selinux/mdp/mdp<br>
  HOSTCC  scripts/dtc/livetree.o<br>
  CC      scripts/mod/devicetable-offsets.s<br>
  HOSTCC  scripts/dtc/treesource.o<br>
  HOSTCC  scripts/dtc/srcpos.o<br>
  HOSTCC  scripts/dtc/checks.o<br>
  HOSTCC  scripts/dtc/util.o<br>
  MKELF   scripts/mod/elfconfig.h<br>
  GEN     scripts/mod/devicetable-offsets.h<br>
  HOSTCC  scripts/mod/sumversion.o<br>
  SHIPPED scripts/dtc/dtc-lexer.lex.c<br>
  SHIPPED scripts/dtc/dtc-parser.tab.h<br>
  SHIPPED scripts/dtc/dtc-parser.tab.c<br>
  HOSTCC  scripts/dtc/dtc-lexer.lex.o<br>
  HOSTCC  scripts/dtc/dtc-parser.tab.o<br>
  HOSTCC  scripts/mod/modpost.o<br>
  HOSTCC  scripts/mod/file2alias.o<br>
  HOSTLD  scripts/genksyms/genksyms<br>
  HOSTLD  scripts/dtc/dtc<br>
  HOSTLD  scripts/mod/modpost<br>
  CC      kernel/bounds.s<br>
  GEN     include/generated/bounds.h<br>
  CC      arch/arm64/kernel/asm-offsets.s<br>
  GEN     include/generated/asm-offsets.h<br>
  CALL    /home/stalker/hadk/kernel/xiaomi/msm8953/scripts/checksyscalls.sh<br>
  HOSTCC  usr/gen_init_cpio<br>
  CC      init/main.o<br>
  CHK     include/generated/compile.h<br>
  CC      init/do_mounts.o<br>
  CC      init/do_mounts_rd.o<br>
  CC      init/do_mounts_initrd.o<br>
  CC      init/do_mounts_dm.o<br>
  CC      init/noinitramfs.o<br>
  CC      init/initramfs.o<br>
  CC      init/calibrate.o<br>
  CC      init/init_task.o<br>
  CC      arch/arm64/mm/dma-mapping.o<br>
  CC      arch/arm64/mm/extable.o<br>
  CC      arch/arm64/mm/fault.o<br>
  GEN     usr/initramfs_data.cpio.gz<br>
  CC      arch/arm64/crypto/sha1-ce-glue.o<br>
  AS      usr/initramfs_data.o<br>
  LD      usr/built-in.o<br>
  CC      arch/arm64/mm/init.o<br>
  AS      arch/arm64/kernel/head.o<br>
  LDS     arch/arm64/kernel/vmlinux.lds<br>
  LDS     arch/arm64/kernel/vdso/vdso.lds<br>
  VDSOA   arch/arm64/kernel/vdso/gettimeofday.o<br>
  CC      init/version.o<br>
  VDSOA   arch/arm64/kernel/vdso/note.o<br>
  VDSOA   arch/arm64/kernel/vdso/sigreturn.o<br>
  AS      arch/arm64/crypto/sha1-ce-core.o<br>
  VDSOL   arch/arm64/kernel/vdso/vdso.so.dbg<br>
  CC      arch/arm64/crypto/sha2-ce-glue.o<br>
  OBJCOPY arch/arm64/kernel/vdso/vdso.so<br>
  VDSOSYM arch/arm64/kernel/vdso/vdso-offsets.h<br>
  AS      arch/arm64/kernel/vdso/vdso.o<br>
  AS      arch/arm64/crypto/sha2-ce-core.o<br>
  LD      arch/arm64/kernel/vdso/built-in.o<br>
  AS      arch/arm64/mm/cache.o<br>
  CC      arch/arm64/kernel/cputable.o<br>
  CC      arch/arm64/kernel/debug-monitors.o<br>
  AS      arch/arm64/kernel/entry.o<br>
  CC      arch/arm64/kernel/irq.o<br>
  CC      arch/arm64/kernel/fpsimd.o<br>
  AS      arch/arm64/kernel/entry-fpsimd.o<br>
  CC      arch/arm64/crypto/ghash-ce-glue.o<br>
  CC      arch/arm64/mm/copypage.o<br>
  AS      arch/arm64/crypto/ghash-ce-core.o<br>
  CC      arch/arm64/kernel/process.o<br>
  CC      arch/arm64/kernel/ptrace.o<br>
  CC      kernel/fork.o<br>
  CC      arch/arm64/kernel/setup.o<br>
  CC      arch/arm64/crypto/aes-ce-cipher.o<br>
  CC      arch/arm64/kernel/signal.o<br>
  CC      kernel/exec_domain.o<br>
  CC      arch/arm64/crypto/aes-ce-ccm-glue.o<br>
  CC      arch/arm64/kernel/sys.o<br>
  AS      arch/arm64/crypto/aes-ce-ccm-core.o<br>
  CC      arch/arm64/crypto/aes-glue-ce.o<br>
  CC      kernel/panic.o<br>
  LD      init/mounts.o<br>
  LD      init/built-in.o<br>
  AS      arch/arm64/crypto/aes-ce.o<br>
  CC      kernel/cpu.o<br>
  CC      arch/arm64/mm/flush.o<br>
  CC      arch/arm64/crypto/aes-glue-neon.o<br>
  AS      arch/arm64/crypto/aes-neon.o<br>
  CC      arch/arm64/mm/ioremap.o<br>
  CC      arch/arm64/mm/mmap.o<br>
  CC      arch/arm64/kernel/stacktrace.o<br>
  CC      arch/arm64/kernel/time.o<br>
  CC      mm/filemap.o<br>
  CC      arch/arm64/mm/pgd.o<br>
  CC      mm/mempool.o<br>
  CC      arch/arm64/kernel/traps.o<br>
  CC      mm/oom_kill.o<br>
  CC      mm/maccess.o<br>
  CC      mm/page_alloc.o<br>
  CC      arch/arm64/mm/mmu.o<br>
  CC      arch/arm64/crypto/crc32-arm64.o<br>
  CC      kernel/exit.o<br>
  CC      kernel/softirq.o<br>
  CC      kernel/resource.o<br>
  CC      fs/open.o<br>
  CC      kernel/sysctl.o<br>
  CC      arch/arm64/kernel/io.o<br>
  CC      kernel/sysctl_binary.o<br>
  CC      arch/arm64/mm/context.o<br>
  CC      mm/page-writeback.o<br>
  CC      arch/arm64/kernel/vdso.o<br>
  CC      fs/read_write.o<br>
  LD      arch/arm64/crypto/sha1-ce.o<br>
  LD      arch/arm64/crypto/sha2-ce.o<br>
  LD      arch/arm64/crypto/ghash-ce.o<br>
  LD      arch/arm64/crypto/aes-ce-ccm.o<br>
  LD      arch/arm64/crypto/aes-ce-blk.o<br>
  LD      arch/arm64/crypto/aes-neon-blk.o<br>
  LD      arch/arm64/crypto/built-in.o<br>
  AS      arch/arm64/mm/proc.o<br>
  CC      ipc/compat.o<br>
  CC      ipc/util.o<br>
  CC      ipc/msgutil.o<br>
  CC      security/integrity/iint.o<br>
  AS      arch/arm64/kernel/hyp-stub.o<br>
  CC      arch/arm64/mm/pageattr.o<br>
  CC      arch/arm64/kernel/psci.o<br>
  CC      ipc/msg.o<br>
  CC      kernel/capability.o<br>
  CC      kernel/ptrace.o<br>
  CC      ipc/sem.o<br>
  CC      security/integrity/integrity_audit.o<br>
  CC      ipc/shm.o<br>
  CC      ipc/ipcns_notifier.o<br>
  LD      arch/arm64/mm/built-in.o<br>
  AS      arch/arm64/kernel/psci-call.o<br>
  CC      arch/arm64/kernel/cpu_ops.o<br>
  CC      arch/arm64/kernel/insn.o<br>
  CC      security/keys/gc.o<br>
  CC      arch/arm64/kernel/return_address.o<br>
  CC      security/keys/key.o<br>
  LD      security/integrity/integrity.o<br>
  CC      kernel/user.o<br>
  LD      security/integrity/built-in.o<br>
  CC      kernel/signal.o<br>
  CC      ipc/syscall.o<br>
  CC      fs/file_table.o<br>
  CC      security/keys/keyring.o<br>
  GEN     security/selinux/flask.h security/selinux/av_permissions.h<br>
  CC      security/selinux/avc.o<br>
  CC      block/bio.o<br>
  CC      block/elevator.o<br>
  CC      kernel/sys.o<br>
  CC      mm/readahead.o<br>
  CC      crypto/api.o<br>
  CC      mm/swap.o<br>
  CC      security/selinux/hooks.o<br>
  CC      arch/arm64/kernel/cpuinfo.o<br>
  CC      security/selinux/selinuxfs.o<br>
  CC      ipc/ipc_sysctl.o<br>
  CC      fs/super.o<br>
  CC      security/commoncap.o<br>
  CC      drivers/amba/bus.o<br>
  CC      arch/arm64/kernel/cpu_errata.o<br>
  CC      security/keys/keyctl.o<br>
  CC      ipc/namespace.o<br>
  CC      drivers/base/component.o<br>
  CC      arch/arm64/kernel/cpufeature.o<br>
  CC      drivers/base/core.o<br>
  CC      drivers/base/bus.o<br>
  LD      ipc/built-in.o<br>
  CC      crypto/cipher.o<br>
  CC      crypto/compress.o<br>
  LD      drivers/amba/built-in.o<br>
  CC      crypto/memneq.o<br>
  CC      crypto/crypto_wq.o<br>
  CC      security/min_addr.o<br>
  CC      mm/truncate.o<br>
  CC      block/blk-core.o<br>
  CC      arch/arm64/kernel/alternative.o<br>
  CC      security/keys/permission.o<br>
  CC      fs/char_dev.o<br>
  CC      mm/vmscan.o<br>
  CC      crypto/algapi.o<br>
  CC      kernel/kmod.o<br>
  CC      sound/sound_core.o<br>
  AS      arch/arm64/kernel/sys32.o<br>
  AS      arch/arm64/kernel/kuser32.o<br>
  CC      arch/arm64/kernel/signal32.o<br>
  CC      arch/arm64/kernel/sys_compat.o<br>
  CC      mm/shmem.o<br>
  CC      kernel/workqueue.o<br>
  CC      kernel/pid.o<br>
  CC      sound/core/sound.o<br>
  CC      security/keys/process_keys.o<br>
  CC      sound/core/init.o<br>
  CC      fs/stat.o<br>
  CC      security/security.o<br>
  CC      arch/arm64/kernel/../../arm/kernel/opcodes.o<br>
  CC      drivers/base/dd.o<br>
  CC      arch/arm64/kernel/ftrace.o<br>
  CC      crypto/scatterwalk.o<br>
  CC      security/keys/request_key.o<br>
  CC      crypto/proc.o<br>
  CC      security/selinux/netlink.o<br>
  CC      crypto/aead.o<br>
  CC      crypto/ablkcipher.o<br>
  CC      security/selinux/nlmsgtab.o<br>
  AS      arch/arm64/kernel/entry-ftrace.o<br>
  CC      arch/arm64/kernel/arm64ksyms.o<br>
  CC      sound/core/memory.o<br>
  CC      fs/exec.o<br>
  CC      drivers/base/syscore.o<br>
  CC      sound/core/info.o<br>
  CC      security/keys/request_key_auth.o<br>
  CC      crypto/blkcipher.o<br>
  CC      security/selinux/netif.o<br>
  CC      sound/core/control.o<br>
  CC      security/selinux/netnode.o<br>
  CC      security/selinux/netport.o<br>
  CC      arch/arm64/kernel/module.o<br>
  CC      block/blk-tag.o<br>
  CC      security/keys/user_defined.o<br>
  CC      crypto/chainiv.o<br>
  CC      mm/util.o<br>
  CC      security/keys/compat.o<br>
  CC      drivers/base/driver.o<br>
  CC      arch/arm64/kernel/smp.o<br>
  CC      kernel/task_work.o<br>
  CC      security/capability.o<br>
  CC      security/selinux/exports.o<br>
  CC      drivers/base/class.o<br>
  CC      security/keys/proc.o<br>
  CC      security/keys/sysctl.o<br>
  CC      security/selinux/ss/ebitmap.o<br>
  CC      block/blk-sysfs.o<br>
  CC      security/selinux/ss/hashtab.o<br>
  CC      security/keys/encrypted-keys/encrypted.o<br>
  CC      fs/pipe.o<br>
  CC      kernel/extable.o<br>
  CC      crypto/eseqiv.o<br>
  CC      security/keys/encrypted-keys/ecryptfs_format.o<br>
  CC      security/selinux/ss/symtab.o<br>
  CC      security/lsm_audit.o<br>
  CC      mm/mmzone.o<br>
  CC      mm/vmstat.o<br>
  CC      mm/backing-dev.o<br>
  CC      security/selinux/ss/sidtab.o<br>
  CC      arch/arm64/kernel/smp_spin_table.o<br>
  CC      kernel/params.o<br>
  CC      sound/core/misc.o<br>
  CC      arch/arm64/kernel/topology.o<br>
  CC      drivers/base/platform.o<br>
  CC      drivers/base/cpu.o<br>
  CC      drivers/block/brd.o<br>
  CC      crypto/seqiv.o<br>
  CC      block/blk-flush.o<br>
  CC      drivers/block/loop.o<br>
  CC      security/device_cgroup.o<br>
  LD      security/keys/encrypted-keys/encrypted-keys.o<br>
  LD      security/keys/encrypted-keys/built-in.o<br>
  LD      security/keys/built-in.o<br>
  CC      mm/mm_init.o<br>
  CC      sound/core/device.o<br>
  CC      fs/namei.o<br>
  CC      fs/fcntl.o<br>
  CC      arch/arm64/kernel/perf_regs.o<br>
  CC      fs/ioctl.o<br>
  CC      sound/core/jack.o<br>
  CC      security/selinux/ss/avtab.o<br>
  CC      crypto/ahash.o<br>
  CC      fs/readdir.o<br>
  CC      kernel/kthread.o<br>
  CC      fs/select.o<br>
  CC      mm/mmu_context.o<br>
  CC      fs/dcache.o<br>
  CC      drivers/base/firmware.o<br>
  CC      block/blk-settings.o<br>
  CC      arch/arm64/kernel/perf_event.o<br>
  CC      block/blk-ioc.o<br>
  CC      mm/percpu.o<br>
  CC      sound/core/hwdep.o<br>
  CC      drivers/base/init.o<br>
  CC      drivers/base/map.o<br>
  CC      fs/inode.o<br>
  CC      sound/core/timer.o<br>
  CC      security/selinux/ss/policydb.o<br>
  CC      kernel/sys_ni.o<br>
  CC      crypto/shash.o<br>
  CC      drivers/base/devres.o<br>
  CC      kernel/nsproxy.o<br>
  CC      drivers/base/attribute_container.o<br>
  CC      block/blk-map.o<br>
  CC      drivers/block/zram/zcomp_lzo.o<br>
  CC      kernel/notifier.o<br>
  CC      drivers/block/zram/zcomp.o<br>
  CC      sound/core/pcm.o<br>
  CC      arch/arm64/kernel/perf_debug.o<br>
  CC      sound/core/pcm_native.o<br>
  CC      fs/attr.o<br>
  CC      mm/slab_common.o<br>
  CC      drivers/base/transport_class.o<br>
  CC      kernel/ksysfs.o<br>
  CC      drivers/block/zram/zram_drv.o<br>
  CC      block/blk-exec.o<br>
  CC      arch/arm64/kernel/perf_trace_counters.o<br>
  CC      sound/core/pcm_lib.o<br>
  CC      fs/bad_inode.o<br>
  CC      fs/file.o<br>
  CC      fs/filesystems.o<br>
  CC      crypto/pcompress.o<br>
  CC      sound/core/pcm_timer.o<br>
  CC      kernel/cred.o<br>
  LD      drivers/bluetooth/built-in.o<br>
  CC      drivers/base/topology.o<br>
  CC      kernel/reboot.o<br>
  CC      kernel/async.o<br>
  CC      arch/arm64/kernel/perf_trace_user.o<br>
  CC      block/blk-merge.o<br>
  CC      drivers/block/zram/zcomp_lz4.o<br>
  CC      security/selinux/ss/services.o<br>
  CC      sound/core/pcm_misc.o<br>
  CC      drivers/base/container.o<br>
  CC      fs/namespace.o<br>
  LD      drivers/block/zram/zram.o<br>
  LD      drivers/block/zram/built-in.o<br>
  CC      fs/seq_file.o<br>
  LD      drivers/block/built-in.o<br>
  CC      security/selinux/ss/conditional.o<br>
  CC      crypto/algboss.o<br>
  CC      crypto/testmgr.o<br>
  AS      arch/arm64/kernel/sleep.o<br>
  CC      drivers/base/property.o<br>
  CC      fs/xattr.o<br>
  CC      arch/arm64/kernel/suspend.o<br>
  CC      kernel/range.o<br>
  CC      drivers/char/mem.o<br>
  CC      kernel/groups.o<br>
  CC      fs/libfs.o<br>
  CC      mm/compaction.o<br>
  CC      arch/arm64/kernel/cpuidle.o<br>
  CC      block/blk-softirq.o<br>
  CC      drivers/base/devtmpfs.o<br>
  CC      drivers/char/random.o<br>
  CC      drivers/base/dma-contiguous.o<br>
  CC      crypto/cmac.o<br>
  CC      arch/arm64/kernel/efi.o<br>
  CC      crypto/hmac.o<br>
  CC      drivers/char/misc.o<br>
  CC      drivers/char/msm_smd_pkt.o<br>
  CC      sound/core/pcm_memory.o<br>
  CC      kernel/smpboot.o<br>
  CC      drivers/base/power/sysfs.o<br>
  CC      drivers/base/power/generic_ops.o<br>
  CC      kernel/bpf/core.o<br>
  CC      drivers/base/power/common.o<br>
  CC      arch/arm64/kernel/efi-stub.o<br>
  CC      block/blk-timeout.o<br>
  CC      security/selinux/ss/mls.o<br>
  AS      arch/arm64/kernel/efi-entry.o<br>
  CC      block/blk-iopoll.o<br>
  CC      crypto/xcbc.o<br>
  CC      arch/arm64/kernel/pci.o<br>
  CC      mm/vmacache.o<br>
  CC      sound/core/memalloc.o<br>
  CC      mm/interval_tree.o<br>
  CC      security/selinux/ss/status.o<br>
  CC      fs/fs-writeback.o<br>
  CC      drivers/base/power/qos.o<br>
  CC      drivers/base/power/runtime.o<br>
  CC      arch/arm64/kernel/armv8_deprecated.o<br>
  CC      drivers/base/power/main.o<br>
  CC      mm/list_lru.o<br>
  CC      crypto/crypto_null.o<br>
  CC      mm/workingset.o<br>
  CC      block/blk-lib.o<br>
  CC      drivers/clk/clk-devres.o<br>
  CC      block/blk-mq.o<br>
  LD      kernel/bpf/built-in.o<br>
  CC      sound/core/rawmidi.o<br>
  CC      kernel/events/core.o<br>
  CC      drivers/char/diag/diagchar_core.o<br>
  CC      crypto/md4.o<br>
  CC      drivers/char/diag/diagchar_hdlc.o<br>
  CC      drivers/char/diag/diagfwd.o<br>
  LD      security/selinux/selinux.o<br>
  LD      security/selinux/built-in.o<br>
  LD      security/built-in.o<br>
  CC      drivers/char/diag/diagfwd_peripheral.o<br>
  CC      mm/iov_iter.o<br>
  LD      arch/arm64/kernel/built-in.o<br>
  CC      drivers/clk/clkdev.o<br>
  CC      drivers/base/power/wakeup.o<br>
  CC      net/socket.o<br>
  CC      drivers/char/diag/diagfwd_smd.o<br>
  CC      net/802/p8022.o<br>
  CC      drivers/char/hw_random/core.o<br>
  CC      crypto/md5.o<br>
  CC      drivers/clk/clk.o<br>
  CC      net/802/psnap.o<br>
  CC      drivers/char/diag/diagfwd_socket.o<br>
  CC      crypto/sha1_generic.o<br>
  CC      drivers/char/hw_random/msm_rng.o<br>
  CC      mm/debug.o<br>
  CC      mm/fremap.o<br>
  CC      mm/gup.o<br>
  CC      sound/core/compress_offload.o<br>
  CC      mm/highmem.o<br>
  CC      fs/pnode.o<br>
  CC      drivers/base/power/opp/core.o<br>
  CC      drivers/clk/msm/clock.o<br>
  CC      block/blk-mq-tag.o<br>
  CC      block/blk-mq-sysfs.o<br>
  CC      crypto/sha256_generic.o<br>
  LD      drivers/char/hw_random/rng-core.o<br>
  LD      drivers/char/hw_random/built-in.o<br>
  CC      crypto/sha512_generic.o<br>
  CC      drivers/base/power/clock_ops.o<br>
  CC      net/802/stp.o<br>
  CC      fs/splice.o<br>
  CC      drivers/clocksource/clksrc-of.o<br>
  CC      fs/sync.o<br>
  CC      mm/memory.o<br>
  CC      drivers/char/diag/diag_mux.o<br>
  LD      sound/core/snd.o<br>
  LD      sound/core/snd-hwdep.o<br>
  CC      mm/mincore.o<br>
  LD      sound/core/snd-timer.o<br>
  LD      sound/core/snd-pcm.o<br>
  CC      net/8021q/vlan_core.o<br>
  LD      sound/core/snd-rawmidi.o<br>
  CC      drivers/clocksource/arm_arch_timer.o<br>
  LD      sound/core/snd-compress.o<br>
  LD      sound/core/built-in.o<br>
  CC      block/blk-mq-cpu.o<br>
  CC      drivers/base/power/opp/cpu.o<br>
  CC      drivers/char/diag/diag_memorydevice.o<br>
  CC      net/8021q/vlan.o<br>
  LD      net/802/built-in.o<br>
  CC      net/8021q/vlan_dev.o<br>
  CC      drivers/clk/msm/clock-dummy.o<br>
  CC      kernel/events/ring_buffer.o<br>
  CC      crypto/gf128mul.o<br>
  LD      net/bluetooth/built-in.o<br>
  LD      net/bluetooth/bnep/built-in.o<br>
  CC      net/bridge/br.o<br>
  CC      block/blk-mq-cpumap.o<br>
  LD      drivers/base/power/opp/built-in.o<br>
  LD      net/bluetooth/hidp/built-in.o<br>
  CC      drivers/base/power/boeffla_wl_blocker.o<br>
  CC      drivers/clocksource/dummy_timer.o<br>
  LD      net/bluetooth/rfcomm/built-in.o<br>
  CC      drivers/char/diag/diag_usb.o<br>
  CC      net/bridge/br_device.o<br>
  CC      drivers/clk/msm/clock-generic.o<br>
  CC      fs/utimes.o<br>
  CC      net/8021q/vlan_netlink.o<br>
  CC      fs/stack.o<br>
  CC      kernel/events/callchain.o<br>
  LD      drivers/clocksource/built-in.o<br>
  LD      drivers/base/power/built-in.o<br>
  CC      net/8021q/vlanproc.o<br>
  CC      drivers/coresight/coresight.o<br>
  CC      drivers/base/regmap/regmap.o<br>
  CC      sound/soc/soc-core.o<br>
  CC      block/ioctl.o<br>
  CC      drivers/base/regmap/regcache.o<br>
  CC      drivers/char/diag/diagmem.o<br>
  CC      drivers/char/diag/diagfwd_cntl.o<br>
  CC      drivers/char/diag/diag_dci.o<br>
  CC      crypto/ecb.o<br>
  CC      fs/fs_struct.o<br>
  CC      drivers/clk/msm/clock-local2.o<br>
  LD      kernel/events/built-in.o<br>
  CC      mm/mlock.o<br>
  CC      net/bridge/br_fdb.o<br>
  CC      net/bridge/br_forward.o<br>
  CC      kernel/irq/irqdesc.o<br>
  CC      kernel/irq/handle.o<br>
  LD      net/8021q/8021q.o<br>
  LD      net/8021q/built-in.o<br>
  CC      mm/mmap.o<br>
  CC      drivers/coresight/coresight-event.o<br>
  CC      block/genhd.o<br>
  CC      crypto/cbc.o<br>
  CC      fs/statfs.o<br>
  CC      crypto/cts.o<br>
  CC      drivers/char/diag/diag_masks.o<br>
  CC      net/bridge/br_if.o<br>
  CC      kernel/irq/manage.o<br>
  CC      drivers/coresight/coresight-fuse.o<br>
  CC      drivers/coresight/coresight-cti.o<br>
  CC      drivers/char/diag/diag_debugfs.o<br>
  CC      net/bridge/br_input.o<br>
  CC      kernel/irq/spurious.o<br>
  CC      drivers/base/regmap/regcache-rbtree.o<br>
  CC      crypto/xts.o<br>
  CC      drivers/clk/msm/clock-pll.o<br>
  CC      fs/fs_pin.o<br>
  CC      drivers/clk/msm/clock-alpha-pll.o<br>
  CC      drivers/base/regmap/regcache-lzo.o<br>
  CC      net/bridge/br_ioctl.o<br>
  CC      block/scsi_ioctl.o<br>
  CC      drivers/base/regmap/regcache-flat.o<br>
  CC      kernel/irq/resend.o<br>
  CC      fs/buffer.o<br>
  CC      kernel/irq/chip.o<br>
  LD      drivers/char/diag/diagchar.o<br>
  LD      drivers/char/diag/built-in.o<br>
  CC      drivers/char/adsprpc.o<br>
  CC      mm/mprotect.o<br>
  CC      fs/block_dev.o<br>
  CC      drivers/coresight/coresight-csr.o<br>
  CC      crypto/ctr.o<br>
  CC      drivers/clk/msm/clock-rpm.o<br>
  CC      drivers/clk/msm/clock-voter.o<br>
  CC      drivers/clk/msm/clock-pm.o<br>
  CC      drivers/base/regmap/regmap-debugfs.o<br>
  CC      drivers/clk/msm/msm-clock-controller.o<br>
  CC      drivers/base/regmap/regmap-i2c.o<br>
  CC      sound/soc/soc-dapm.o<br>
  CC      net/bridge/br_stp.o<br>
  CC      drivers/base/regmap/regmap-spi.o<br>
  CC      drivers/coresight/coresight-tmc.o<br>
  CC      drivers/coresight/coresight-tpiu.o<br>
  CC      drivers/clk/msm/clock-debug.o<br>
  CC      crypto/cryptd.o<br>
  CC      mm/mremap.o<br>
  CC      kernel/irq/dummychip.o<br>
  CC      block/partition-generic.o<br>
  CC      block/ioprio.o<br>
  CC      drivers/clk/msm/clock-a7.o<br>
  AS      arch/arm64/lib/bitops.o<br>
  AS      arch/arm64/lib/clear_page.o<br>
  CC      kernel/irq/devres.o<br>
  AS      arch/arm64/lib/clear_user.o<br>
  AS      arch/arm64/lib/copy_from_user.o<br>
  CC      drivers/base/regmap/regmap-swr.o<br>
  AS      arch/arm64/lib/copy_in_user.o<br>
  CC      fs/direct-io.o<br>
  AS      arch/arm64/lib/copy_page.o<br>
  AS      arch/arm64/lib/copy_to_user.o<br>
  CC      fs/mpage.o<br>
  CC      arch/arm64/lib/delay.o<br>
  CC      mm/msync.o<br>
  CC      net/bridge/br_stp_bpdu.o<br>
  CC      fs/proc_namespace.o<br>
  CC      drivers/clk/msm/clock-cpu-8939.o<br>
  CC      drivers/char/adsprpc_compat.o<br>
  CC      drivers/clk/msm/clock-gcc-8952.o<br>
  CC      kernel/irq/autoprobe.o<br>
  CC      crypto/des_generic.o<br>
  CC      block/partitions/check.o<br>
  AS      arch/arm64/lib/memchr.o<br>
  CC      mm/rmap.o<br>
  AS      arch/arm64/lib/memcmp.o<br>
  LD      drivers/base/regmap/built-in.o<br>
  CC      drivers/coresight/coresight-nidnt.o<br>
  CC      drivers/base/dma-mapping.o<br>
  AS      arch/arm64/lib/memcpy.o<br>
  AS      arch/arm64/lib/memmove.o<br>
  CC      block/bounce.o<br>
  AS      arch/arm64/lib/memset.o<br>
  AS      arch/arm64/lib/strchr.o<br>
  AS      arch/arm64/lib/strcmp.o<br>
  AS      arch/arm64/lib/strlen.o<br>
  AS      arch/arm64/lib/strncmp.o<br>
  AS      arch/arm64/lib/strnlen.o<br>
  AS      arch/arm64/lib/strrchr.o<br>
  CC      net/bridge/br_stp_if.o<br>
  AS      arch/arm64/lib/tishift.o<br>
  AR      arch/arm64/lib/lib.a<br>
  CC      kernel/irq/irqdomain.o<br>
  LD      fs/autofs4/built-in.o<br>
  CC      net/bridge/br_stp_timer.o<br>
  CC      drivers/char/frandom.o<br>
  CC      block/partitions/msdos.o<br>
  CC      block/partitions/efi.o<br>
  CC      drivers/cpufreq/cpufreq.o<br>
  CC      net/core/sock.o<br>
  CC      drivers/clk/msm/clock-gcc-8953.o<br>
  CC      sound/soc/soc-jack.o<br>
  CC      drivers/base/dma-coherent.o<br>
  CC      drivers/coresight/coresight-funnel.o<br>
  CC      drivers/coresight/coresight-replicator.o<br>
  CC      crypto/twofish_generic.o<br>
  CC      crypto/twofish_common.o<br>
  CC      mm/vmalloc.o<br>
  LD      block/partitions/built-in.o<br>
  CC      block/bsg.o<br>
  LD      drivers/char/built-in.o<br>
  CC      net/bridge/br_netlink.o<br>
  CC      drivers/cpuidle/cpuidle.o<br>
  CC      mm/pagewalk.o<br>
  CC      drivers/cpuidle/driver.o<br>
  CC      fs/cifs/cifsfs.o<br>
  CC      kernel/irq/proc.o<br>
  CC      drivers/clk/msm/clock-rcgwr.o<br>
  CC      drivers/coresight/coresight-stm.o<br>
  CC      drivers/base/dma-removed.o<br>
  CC      sound/soc/soc-cache.o<br>
  CC      sound/soc/soc-utils.o<br>
  CC      crypto/aes_generic.o<br>
  CC      drivers/cpuidle/governor.o<br>
  CC      kernel/irq/pm.o<br>
  CC      drivers/clk/msm/clock-cpu-8953.o<br>
  CC      kernel/irq/msi.o<br>
  CC      net/bridge/br_sysfs_if.o<br>
  CC      drivers/coresight/coresight-hwevent.o<br>
  CC      drivers/cpuidle/sysfs.o<br>
  CC      drivers/cpufreq/freq_table.o<br>
  CC      drivers/base/firmware_class.o<br>
  CC      block/blk-cgroup.o<br>
  CC      block/noop-iosched.o<br>
  CC      block/deadline-iosched.o<br>
  CC      sound/soc/soc-pcm.o<br>
  CC      fs/cifs/cifssmb.o<br>
  CC      fs/cifs/cifs_debug.o<br>
  LD      drivers/coresight/built-in.o<br>
  CC      drivers/clk/msm/gdsc.o<br>
  CC      fs/cifs/connect.o<br>
  LD      kernel/irq/built-in.o<br>
  CC      mm/pgtable-generic.o<br>
  CC      drivers/cpuidle/governors/ladder.o<br>
  CC      crypto/arc4.o<br>
  CC      drivers/cpufreq/cpufreq_stats.o<br>
  CC      kernel/locking/mutex.o<br>
  CC      crypto/deflate.o<br>
  CC      net/core/request_sock.o<br>
  CC      kernel/power/qos.o<br>
  CC      net/bridge/br_sysfs_br.o<br>
  CC      mm/process_vm_access.o<br>
  CC      drivers/cpuidle/governors/menu.o<br>
  CC      mm/showmem.o<br>
  CC      crypto/crc32c_generic.o<br>
  CC      block/cfq-iosched.o<br>
  CC      drivers/base/module.o<br>
  CC      block/zen-iosched.o<br>
  CC      drivers/clk/msm/mdss/mdss-pll-util.o<br>
  CC      kernel/locking/semaphore.o<br>
  LD      drivers/cpuidle/governors/built-in.o<br>
  CC      drivers/cpuidle/lpm-levels.o<br>
  CC      kernel/locking/rwsem.o<br>
  CC      drivers/base/soc.o<br>
  CC      mm/vmpressure.o<br>
  CC      crypto/crc32.o<br>
  CC      net/core/skbuff.o<br>
  CC      drivers/cpufreq/cpufreq_performance.o<br>
  CC      drivers/cpufreq/cpufreq_powersave.o<br>
  CC      drivers/clk/msm/mdss/mdss-pll.o<br>
  CC      kernel/power/main.o<br>
  CC      net/bridge/br_nf_core.o<br>
  CC      sound/soc/soc-compress.o<br>
  CC      kernel/locking/mcs_spinlock.o<br>
  CC      net/ethernet/eth.o<br>
  CC      drivers/base/pinctrl.o<br>
  CC      crypto/authenc.o<br>
  CC      mm/init-mm.o<br>
  CC      mm/nobootmem.o<br>
  CC      drivers/cpufreq/cpufreq_userspace.o<br>
  CC      kernel/locking/spinlock.o<br>
  CC      kernel/locking/lglock.o<br>
  CC      drivers/base/devcoredump.o<br>
  CC      drivers/clk/msm/mdss/mdss-dsi-pll-util.o<br>
  CC      kernel/locking/rtmutex.o<br>
  CC      drivers/cpufreq/cpufreq_ondemand.o<br>
  CC      fs/cifs/dir.o<br>
  CC      net/bridge/br_multicast.o<br>
  CC      kernel/power/console.o<br>
  CC      sound/soc/soc-io.o<br>
  CC      block/fiops-iosched.o<br>
  CC      block/sio-iosched.o<br>
  CC      mm/fadvise.o<br>
  CC      mm/madvise.o<br>
  LD      drivers/base/built-in.o<br>
  CC      mm/memblock.o<br>
  CC      drivers/clk/msm/mdss/mdss-dsi-pll-28lpm.o<br>
  CC      kernel/locking/rwsem-xadd.o<br>
  LD      net/ethernet/built-in.o<br>
  CC      drivers/cpuidle/lpm-levels-of.o<br>
  CC      drivers/cpuidle/lpm-workarounds.o<br>
  CC      crypto/authencesn.o<br>
  CC      kernel/power/process.o<br>
  CC      crypto/lzo.o<br>
  CC      drivers/cpufreq/cpufreq_zzmoove.o<br>
  CC      block/maple-iosched.o<br>
  CC      sound/soc/soc-devres.o<br>
  CC      drivers/clk/msm/mdss/mdss-dsi-pll-8996.o<br>
  CC      crypto/rng.o<br>
  CC      fs/cifs/file.o<br>
  LD      kernel/locking/built-in.o<br>
  CC      net/bridge/br_mdb.o<br>
  CC      lib/lockref.o<br>
  CC      mm/page_io.o<br>
  CC      drivers/cpufreq/cpufreq_interactive.o<br>
  CC      kernel/printk/printk.o<br>
  CC      lib/bcd.o<br>
  CC      drivers/clk/msm/mdss/mdss-dsi-pll-8996-util.o<br>
  CC      lib/div64.o<br>
  CC      block/iosched_switcher.o<br>
  LD      drivers/cpuidle/built-in.o<br>
  CC      kernel/power/suspend.o<br>
  CC      lib/sort.o<br>
  CC      net/ipc_router/ipc_router_core.o<br>
  CC      sound/usb/card.o<br>
  CC      crypto/krng.o<br>
  CC      mm/swap_state.o<br>
  CC      sound/soc/codecs/msm_hdmi_dba_codec_rx.o<br>
  CC      sound/soc/codecs/wcd9330.o<br>
  CC      block/bfq-iosched.o<br>
  CC      net/bridge/br_netfilter.o<br>
  CC      lib/parser.o<br>
  CC      net/core/iovec.o<br>
  CC      drivers/clk/msm/mdss/mdss-hdmi-pll-8996.o<br>
  CC      crypto/ansi_cprng.o<br>
  CC      lib/halfmd4.o<br>
  CC      mm/swapfile.o<br>
  CC      sound/usb/clock.o<br>
  CC      kernel/power/powersuspend.o<br>
  CC      lib/debug_locks.o<br>
  CC      drivers/cpufreq/cpufreq_blu_active.o<br>
  CC      lib/random32.o<br>
  LD      kernel/printk/built-in.o<br>
  CC      drivers/cpufreq/cpufreq_governor.o<br>
  CC      net/core/datagram.o<br>
  LD      drivers/clk/msm/mdss/built-in.o<br>
  CC      kernel/rcu/update.o<br>
  CC      crypto/asymmetric_keys/asymmetric_type.o<br>
  LD      drivers/clk/msm/built-in.o<br>
  LD      drivers/clk/built-in.o<br>
  CC      crypto/asymmetric_keys/signature.o<br>
  CC      fs/cifs/inode.o<br>
  CC      fs/cifs/link.o<br>
  CC      sound/usb/endpoint.o<br>
  CC      kernel/power/autosleep.o<br>
  CC      lib/bust_spinlocks.o<br>
  CC      net/bridge/netfilter/ebtables.o<br>
  CC      lib/hexdump.o<br>
  CC      sound/last.o<br>
  CC      kernel/power/wakelock.o<br>
  CC      crypto/asymmetric_keys/public_key.o<br>
  CC      net/bridge/netfilter/ebtable_broute.o<br>
  CC      drivers/cpufreq/cpufreq_nightmare.o<br>
  CC      net/ipc_router/ipc_router_socket.o<br>
  CC      kernel/rcu/srcu.o<br>
  CC      lib/kasprintf.o<br>
  CC      block/compat_ioctl.o<br>
  CC      sound/usb/format.o<br>
  CC      lib/bitmap.o<br>
  CC      sound/usb/helper.o<br>
  CC      kernel/power/suspend_time.o<br>
  CC      mm/swap_ratio.o<br>
  CC      crypto/asymmetric_keys/rsa.o<br>
  CC      lib/scatterlist.o<br>
  CC      kernel/rcu/tree.o<br>
  CC      lib/gcd.o<br>
  CC      net/core/stream.o<br>
  CC      sound/soc/codecs/wcd9330-tables.o<br>
  CC      lib/lcm.o<br>
../../../../../../kernel/xiaomi/msm8953/drivers/cpufreq/cpufreq_nightmare.c:643:2: warning: initialization from incompatible pointer type<br>
  .init = nm_init,<br>
  ^<br>
../../../../../../kernel/xiaomi/msm8953/drivers/cpufreq/cpufreq_nightmare.c:643:2: warning: (near initialization for 'nm_dbs_cdata.init')<br>
cc1: warning: unrecognized command line option "-Wno-misleading-indentation"<br>
  CC      fs/cifs/misc.o<br>
  CC      fs/cifs/netmisc.o<br>
  CC      drivers/cpufreq/cpufreq_darkness.o<br>
  CC      kernel/power/poweroff.o<br>
  CC      sound/usb/mixer.o<br>
  CC      fs/cifs/smbencrypt.o<br>
  CC      mm/zcache.o<br>
  CC      fs/configfs/inode.o<br>
  CC      kernel/power/wakeup_reason.o<br>
  LD      block/built-in.o<br>
  ASN.1   crypto/asymmetric_keys/x509-asn1.c<br>
Extracted 203 tokens<br>
Extracted 14 types<br>
Extracted 12 actions<br>
Pass 1<br>
Pass 2<br>
  CC      sound/soc/codecs/wcd9335.o<br>
  ASN.1   crypto/asymmetric_keys/x509_rsakey-asn1.c<br>
Extracted 16 tokens<br>
Extracted 1 types<br>
Extracted 1 actions<br>
Pass 1<br>
Pass 2<br>
  CC      crypto/asymmetric_keys/x509_public_key.o<br>
../../../../../../kernel/xiaomi/msm8953/drivers/cpufreq/cpufreq_darkness.c:304:2: warning: initialization from incompatible pointer type<br>
  .init = dk_init,<br>
  ^<br>
../../../../../../kernel/xiaomi/msm8953/drivers/cpufreq/cpufreq_darkness.c:304:2: warning: (near initialization for 'dk_dbs_cdata.init')<br>
cc1: warning: unrecognized command line option "-Wno-misleading-indentation"<br>
  CC      net/ipc_router/ipc_router_security.o<br>
  CC      drivers/cpufreq/cpufreq_alucard.o<br>
  CC      lib/list_sort.o<br>
  CC      fs/configfs/file.o<br>
  CC      drivers/firmware/efi/libstub/arm-stub.o<br>
  CC      drivers/cpufreq/cpufreq_relaxed.o<br>
  LD      kernel/power/built-in.o<br>
  CC      drivers/firmware/efi/libstub/efi-stub-helper.o<br>
  CC      fs/cifs/transport.o<br>
  LD      net/bridge/netfilter/built-in.o<br>
  LD      net/bridge/bridge.o<br>
  LD      net/bridge/built-in.o<br>
  CC      net/core/scm.o<br>
  DTC     arch/arm64/boot/dts/qcom/msm8953-qrd-sku3.dtb<br>
  CC      fs/cifs/asn1.o<br>
  CC      mm/dmapool.o<br>
  LD      crypto/asymmetric_keys/asymmetric_keys.o<br>
  CC      crypto/asymmetric_keys/x509-asn1.o<br>
  CC      lib/uuid.o<br>
  CC      fs/configfs/dir.o<br>
  CC      sound/usb/mixer_quirks.o<br>
  CC      crypto/asymmetric_keys/x509_rsakey-asn1.o<br>
  CC      lib/flex_array.o<br>
  CC      crypto/asymmetric_keys/x509_cert_parser.o<br>
  CC      drivers/cpufreq/cpufreq_chill.o<br>
  CC      drivers/firmware/efi/libstub/fdt.o<br>
  DTC     arch/arm64/boot/dts/qcom/msm8952-qrd-skum.dtb<br>
  LD      net/ipc_router/built-in.o<br>
../../../../../../kernel/xiaomi/msm8953/drivers/cpufreq/cpufreq_alucard.c:710:2: warning: initialization from incompatible pointer type<br>
  .init = ac_init,<br>
  ^<br>
../../../../../../kernel/xiaomi/msm8953/drivers/cpufreq/cpufreq_alucard.c:710:2: warning: (near initialization for 'ac_dbs_cdata.init')<br>
cc1: warning: unrecognized command line option "-Wno-misleading-indentation"<br>
  CC      drivers/cpufreq/cpufreq_electron.o<br>
  CC      drivers/crypto/msm/qcedev.o<br>
  CC      mm/sparse.o<br>
  CC      lib/iovec.o<br>
  DTC     arch/arm64/boot/dts/qcom/msm8952-cdp.dtb<br>
  LD      crypto/asymmetric_keys/x509_key_parser.o<br>
  CC      lib/clz_ctz.o<br>
  LD      crypto/asymmetric_keys/built-in.o<br>
  CC      crypto/hash_info.o<br>
  CC      fs/cifs/cifs_unicode.o<br>
  CC      fs/cifs/nterr.o<br>
  CC      crypto/ablk_helper.o<br>
  CC      lib/bsearch.o<br>
  AR      drivers/firmware/efi/libstub/lib.a<br>
  CC      lib/find_last_bit.o<br>
  CC      net/ipv4/route.o<br>
  CC      net/core/gen_stats.o<br>
  LD      kernel/rcu/built-in.o<br>
  DTC     arch/arm64/boot/dts/qcom/msm8952-ext-codec-cdp.dtb<br>
  CC      net/core/gen_estimator.o<br>
  CC      fs/crypto/crypto.o<br>
  CC      sound/usb/pcm.o<br>
  CC      lib/find_next_bit.o<br>
  CC      fs/configfs/symlink.o<br>
  CC      kernel/sched/core.o<br>
  CC      lib/llist.o<br>
  CC      kernel/sched/fair.o<br>
  CC      mm/sparse-vmemmap.o<br>
  DTC     arch/arm64/boot/dts/qcom/msm8952-mtp.dtb<br>
  CC      lib/memweight.o<br>
  LD      crypto/crypto.o<br>
  LD      crypto/crypto_algapi.o<br>
  LD      crypto/crypto_blkcipher.o<br>
  LD      crypto/crypto_hash.o<br>
  LD      crypto/cryptomgr.o<br>
  LD      crypto/built-in.o<br>
  CC      fs/configfs/mount.o<br>
  CC      net/ipv4/inetpeer.o<br>
  CC      lib/kfifo.o<br>
  CC      fs/cifs/xattr.o<br>
  CC      mm/ksm.o<br>
  CC      fs/crypto/fname.o<br>
  CC      lib/percpu-refcount.o<br>
  CC      drivers/crypto/msm/qce50.o<br>
  CC      sound/usb/proc.o<br>
  CC      fs/configfs/item.o<br>
  CC      net/core/net_namespace.o<br>
../../../../../../kernel/xiaomi/msm8953/drivers/cpufreq/cpufreq_electron.c: In function 'cpufreq_electron_timer':<br>
../../../../../../kernel/xiaomi/msm8953/drivers/cpufreq/cpufreq_electron.c:469:23: warning: passing argument 1 of 'sched_get_cpus_busy' from incompatible pointer type<br>
   sched_get_cpus_busy(ppol-cpu_busy_times,<br>
                       ^<br>
In file included from ../../../../../../kernel/xiaomi/msm8953/arch/arm64/include/asm/compat.h:25:0,<br>
                 from ../../../../../../kernel/xiaomi/msm8953/arch/arm64/include/asm/stat.h:23,<br>
                 from ../../../../../../kernel/xiaomi/msm8953/include/linux/stat.h:5,<br>
                 from ../../../../../../kernel/xiaomi/msm8953/include/linux/sysfs.h:21,<br>
                 from ../../../../../../kernel/xiaomi/msm8953/include/linux/kobject.h:21,<br>
                 from ../../../../../../kernel/xiaomi/msm8953/include/linux/device.h:17,<br>
                 from ../../../../../../kernel/xiaomi/msm8953/include/linux/node.h:17,<br>
                 from ../../../../../../kernel/xiaomi/msm8953/include/linux/cpu.h:16,<br>
                 from ../../../../../../kernel/xiaomi/msm8953/drivers/cpufreq/cpufreq_electron.c:19:<br>
../../../../../../kernel/xiaomi/msm8953/include/linux/sched.h:2084:13: note: expected 'struct sched_load *' but argument is of type 'long unsigned int *'<br>
 extern void sched_get_cpus_busy(struct sched_load *busy,<br>
             ^<br>
../../../../../../kernel/xiaomi/msm8953/drivers/cpufreq/cpufreq_electron.c: At top level:<br>
cc1: warning: unrecognized command line option "-Wno-misleading-indentation"<br>
  CC      net/ipv6/af_inet6.o<br>
  CC      drivers/cpufreq/fp-boost.o<br>
  CC      net/ipv6/anycast.o<br>
  CC      fs/cifs/cifsencrypt.o<br>
  CC      lib/percpu_ida.o<br>
  CC      fs/crypto/policy.o<br>
  CC      sound/usb/quirks.o<br>
  LD      fs/configfs/configfs.o<br>
  LD      fs/configfs/built-in.o<br>
  CC      drivers/cpufreq/qcom-cpufreq.o<br>
  CC      lib/hash.o<br>
  LD      sound/soundcore.o<br>
  CC      sound/usb/stream.o<br>
  CC      mm/slub.o<br>
  CC      fs/debugfs/inode.o<br>
  CC      lib/rhashtable.o<br>
  CC      sound/soc/codecs/wcdcal-hwdep.o<br>
  CC      net/ipv4/protocol.o<br>
  CC      fs/crypto/keyinfo.o<br>
  CC      net/core/secure_seq.o<br>
  CC      fs/debugfs/file.o<br>
  CC      sound/usb/midi.o<br>
  CC      fs/cifs/readdir.o<br>
  LD      drivers/cpufreq/built-in.o<br>
  CC      net/ipv6/ip6_output.o<br>
  CC      fs/cifs/ioctl.o<br>
  CC      lib/reciprocal_div.o<br>
  CC      net/core/flow_dissector.o<br>
  CC      fs/crypto/bio.o<br>
  CC      lib/string_helpers.o<br>
  CC      sound/soc/codecs/audio-ext-clk.o<br>
  CC      sound/soc/codecs/wcd9xxx-resmgr.o<br>
  CC      drivers/devfreq/devfreq.o<br>
  CC      lib/kstrtox.o<br>
  CC      net/ipv4/ip_input.o<br>
  LD      fs/debugfs/debugfs.o<br>
  LD      fs/debugfs/built-in.o<br>
  CC      fs/devpts/inode.o<br>
  CC      lib/iomap.o<br>
  CC      fs/cifs/sess.o<br>
  CC      drivers/devfreq/devfreq_trace.o<br>
  CC      fs/cifs/export.o<br>
  LD      fs/crypto/fscrypto.o<br>
  LD      fs/crypto/built-in.o<br>
  CC      drivers/crypto/msm/compat_qcedev.o<br>
  CC      sound/soc/codecs/wcd9xxx-mbhc.o<br>
  CC      fs/ecryptfs/dentry.o<br>
  CC      drivers/devfreq/governor_simpleondemand.o<br>
  CC      fs/exfat/exfat_core.o<br>
  CC      net/core/sysctl_net_core.o<br>
  LD      sound/usb/snd-usb-audio.o<br>
  LD      fs/devpts/devpts.o<br>
  LD      fs/devpts/built-in.o<br>
  LD      sound/usb/snd-usbmidi-lib.o<br>
  CC      fs/exfat/exfat_super.o<br>
  LD      sound/usb/built-in.o<br>
  CC      net/core/dev.o<br>
  CC      mm/migrate.o<br>
  CC      lib/pci_iomap.o<br>
  CC      fs/cifs/smb1ops.o<br>
  CC      drivers/devfreq/governor_performance.o<br>
  CC      fs/ecryptfs/file.o<br>
  CC      lib/iomap_copy.o<br>
  CC      net/ipv4/ip_fragment.o<br>
  CC      fs/cifs/winucase.o<br>
  CC      drivers/devfreq/governor_powersave.o<br>
  CC      drivers/devfreq/governor_userspace.o<br>
  CC      lib/devres.o<br>
  CC      lib/hweight.o<br>
  CC      drivers/devfreq/governor_cpufreq.o<br>
  CC      drivers/crypto/msm/qcrypto.o<br>
  CC      net/core/ethtool.o<br>
  CC      fs/ecryptfs/inode.o<br>
  CC      drivers/devfreq/governor_msm_adreno_tz.o<br>
  CC      drivers/devfreq/bimc-bwmon.o<br>
  CC      net/ipv6/ip6_input.o<br>
  CC      net/core/dev_addr_lists.o<br>
  LD      fs/cifs/cifs.o<br>
  LD      fs/cifs/built-in.o<br>
  CC      lib/assoc_array.o<br>
  CC      mm/memcontrol.o<br>
  CC      kernel/sched/rt.o<br>
  CC      mm/page_cgroup.o<br>
  CC      lib/smp_processor_id.o<br>
  CC      drivers/devfreq/arm-memlat-mon.o<br>
  CC      fs/ecryptfs/main.o<br>
  CC      net/ipv4/ip_forward.o<br>
  CC      lib/bitrev.o<br>
  CC      lib/crc-ccitt.o<br>
  CC      lib/crc16.o<br>
  CC      sound/soc/codecs/wcd9xxx-common.o<br>
  CC      net/ipv6/addrconf.o<br>
  CC      fs/exfat/exfat_api.o<br>
  CC      drivers/devfreq/msmcci-hwmon.o<br>
  CC      fs/exfat/exfat_blkdev.o<br>
  CC      fs/exfat/exfat_cache.o<br>
  CC      fs/ecryptfs/super.o<br>
  HOSTCC  lib/gen_crc32table<br>
  CC      lib/libcrc32c.o<br>
  CC      net/ipv4/ip_options.o<br>
  CC      kernel/sched/proc.o<br>
  CC      kernel/sched/clock.o<br>
  CC      net/ipv4/ip_output.o<br>
  CC      drivers/devfreq/m4m-hwmon.o<br>
  CC      fs/ecryptfs/mmap.o<br>
  CC      fs/ecryptfs/read_write.o<br>
  CC      fs/exportfs/expfs.o<br>
  CC      fs/exfat/exfat_data.o<br>
  CC      lib/genalloc.o<br>
  CC      lib/lz4/lz4_compress.o<br>
  CC      fs/ext2/balloc.o<br>
  CC      sound/soc/codecs/wcd9xxx-common-v2.o<br>
  CC      kernel/sched/cputime.o<br>
  CC      drivers/devfreq/governor_bw_hwmon.o<br>
  CC      kernel/sched/idle_task.o<br>
  CC      fs/exfat/exfat_bitmap.o<br>
  CC      fs/ecryptfs/events.o<br>
  CC      fs/exfat/exfat_nls.o<br>
  CC      drivers/crypto/msm/ota_crypto.o<br>
  LD      fs/exportfs/exportfs.o<br>
  LD      fs/exportfs/built-in.o<br>
  CC      lib/lz4/lz4_decompress.o<br>
  CC      sound/soc/codecs/wcd9xxx-resmgr-v2.o<br>
  CC      net/core/dst.o<br>
  CC      fs/ext3/balloc.o<br>
  CC      fs/ext3/bitmap.o<br>
  CC      fs/ext2/dir.o<br>
  CC      fs/ecryptfs/crypto.o<br>
  CC      fs/ext2/file.o<br>
  CC      fs/exfat/exfat_oal.o<br>
  CC      kernel/sched/deadline.o<br>
  LD      lib/lz4/built-in.o<br>
  CC      drivers/devfreq/governor_cache_hwmon.o<br>
  CC      fs/exfat/exfat_upcase.o<br>
  CC      net/core/netevent.o<br>
  CC      lib/lzo/lzo1x_compress.o<br>
  CC      drivers/crypto/msm/ice.o<br>
  CC      sound/soc/codecs/msm8x16-wcd.o<br>
  CC      net/core/neighbour.o<br>
  CC      mm/cleancache.o<br>
  CC      fs/ext2/ialloc.o<br>
  CC      drivers/dma/dmaengine.o<br>
  CC      net/ipv4/ip_sockglue.o<br>
  LD      fs/exfat/exfat.o<br>
  LD      fs/exfat/built-in.o<br>
  CC      drivers/dma/of-dma.o<br>
  CC      net/core/rtnetlink.o<br>
  CC      lib/lzo/lzo1x_decompress_safe.o<br>
  CC      drivers/devfreq/governor_gpubw_mon.o<br>
  CC      fs/ext3/dir.o<br>
  CC      kernel/sched/stop_task.o<br>
  CC      net/ipv6/addrlabel.o<br>
  CC      kernel/sched/wait.o<br>
  CC      fs/ext2/inode.o<br>
  CC      fs/ecryptfs/keystore.o<br>
  CC      mm/page_isolation.o<br>
  CC      drivers/dma/qcom-sps-dma.o<br>
  LD      lib/lzo/lzo_compress.o<br>
  LD      lib/lzo/lzo_decompress.o<br>
  LD      drivers/crypto/msm/built-in.o<br>
  LD      lib/lzo/built-in.o<br>
  CC      drivers/devfreq/governor_bw_vbif.o<br>
  LD      drivers/crypto/built-in.o<br>
  CC      drivers/dma-buf/dma-buf.o<br>
  CC      drivers/dma-buf/fence.o<br>
  CC      mm/zbud.o<br>
  CC      lib/mpi/generic_mpih-lshift.o<br>
  CC      drivers/edac/edac_stub.o<br>
  CC      fs/ext3/file.o<br>
  CC      kernel/sched/completion.o<br>
  CC      drivers/devfreq/governor_spdm_bw_hyp.o<br>
  LD      drivers/dma/built-in.o<br>
  CC      drivers/edac/edac_mc.o<br>
  CC      fs/ext2/ioctl.o<br>
  CC      net/ipv6/route.o<br>
  CC      lib/mpi/generic_mpih-mul1.o<br>
  CC      mm/zsmalloc.o<br>
  CC      fs/ecryptfs/kthread.o<br>
  CC      fs/ext3/fsync.o<br>
  CC      kernel/sched/idle.o<br>
  CC      drivers/edac/edac_device.o<br>
  CC      net/ipv4/inet_hashtables.o<br>
  CC      lib/mpi/generic_mpih-mul2.o<br>
  CC      fs/ext2/namei.o<br>
  CC      drivers/devfreq/governor_memlat.o<br>
  CC      lib/mpi/generic_mpih-mul3.o<br>
  CC      drivers/dma-buf/reservation.o<br>
  CC      fs/ecryptfs/debug.o<br>
  CC      sound/soc/codecs/msm8x16-wcd-tables.o<br>
  CC      drivers/dma-buf/seqno-fence.o<br>
  CC      fs/ext3/ialloc.o<br>
  CC      lib/mpi/generic_mpih-rshift.o<br>
  CC      kernel/sched/sched_avg.o<br>
  CC      lib/mpi/generic_mpih-sub1.o<br>
  CC      fs/ext2/super.o<br>
  CC      net/core/utils.o<br>
  CC      lib/reed_solomon/reed_solomon.o<br>
  LD      fs/ecryptfs/ecryptfs.o<br>
  LD      fs/ecryptfs/built-in.o<br>
  CC      drivers/edac/edac_mc_sysfs.o<br>
  CC      drivers/devfreq/adreno_idler.o<br>
  CC      mm/early_ioremap.o<br>
  CC      sound/soc/codecs/msm8916-wcd-irq.o<br>
  CC      sound/soc/codecs/wcd_cpe_services.o<br>
  CC      sound/soc/codecs/wcd_cpe_core.o<br>
  CC      lib/mpi/generic_mpih-add1.o<br>
  LD      drivers/dma-buf/built-in.o<br>
  HZFILE  kernel/time/hz.bc<br>
  CC      drivers/firmware/dmi_scan.o<br>
  CC      kernel/time/timer.o<br>
  CC      fs/ext3/inode.o<br>
  CC      lib/mpi/mpicoder.o<br>
  CC      mm/cma.o<br>
  CC      drivers/devfreq/devfreq_devbw.o<br>
  CC      net/ipv4/inet_timewait_sock.o<br>
  CC      kernel/sched/cpupri.o<br>
  LD      lib/reed_solomon/built-in.o<br>
  CC      fs/ext3/ioctl.o<br>
  CC      drivers/edac/edac_pci_sysfs.o<br>
  CC      fs/ext2/symlink.o<br>
  CC      fs/ext2/xattr.o<br>
  CC      net/core/link_watch.o<br>
  CC      lib/mpi/mpi-bit.o<br>
  CC      kernel/sched/cpudeadline.o<br>
  CC      net/ipv6/ip6_fib.o<br>
  CC      drivers/devfreq/devfreq_simple_dev.o<br>
  CC      drivers/firmware/dmi-id.o<br>
  CC      net/ipv6/ipv6_sockglue.o<br>
  CC      mm/process_reclaim.o<br>
  CC      drivers/gpio/devres.o<br>
  CC      net/core/filter.o<br>
  CC      kernel/sched/stats.o<br>
  CC      drivers/edac/edac_module.o<br>
  CC      lib/mpi/mpi-cmp.o<br>
  CC      net/ipv4/inet_connection_sock.o<br>
  CC      fs/ext2/xattr_user.o<br>
  CC      drivers/firmware/efi/efi.o<br>
  CC      drivers/devfreq/devfreq_spdm.o<br>
  CC      drivers/gpio/gpiolib.o<br>
  CC      mm/cma_debug.o<br>
  CC      kernel/sched/debug.o<br>
  CC      sound/soc/codecs/wcd-mbhc-v2.o<br>
  CC      kernel/time/hrtimer.o<br>
  CC      drivers/firmware/efi/vars.o<br>
  CC      drivers/edac/edac_device_sysfs.o<br>
  CC      fs/ext2/xattr_trusted.o<br>
  CC      lib/mpi/mpih-cmp.o<br>
  CC      fs/ext3/namei.o<br>
  CC      drivers/devfreq/devfreq_spdm_debugfs.o<br>
  LD      mm/built-in.o<br>
  CC      lib/mpi/mpih-div.o<br>
  CC      fs/ext3/super.o<br>
  LD      fs/ext2/ext2.o<br>
  CC      fs/ext3/symlink.o<br>
  LD      fs/ext2/built-in.o<br>
  CC      net/ipv6/ndisc.o<br>
  CC      drivers/edac/edac_pci.o<br>
  CC      net/core/sock_diag.o<br>
  CC      drivers/firmware/qcom/tz_log.o<br>
  CC      drivers/firmware/efi/reboot.o<br>
  CC      lib/mpi/mpih-mul.o<br>
  CC      kernel/sched/cpuacct.o<br>
  CC      fs/ext3/hash.o<br>
  CC      net/ipv4/tcp.o<br>
  LD      drivers/devfreq/built-in.o<br>
  CC      drivers/gpu/msm/kgsl.o<br>
  CC      fs/ext3/resize.o<br>
  CC      kernel/time/itimer.o<br>
  CC      drivers/gpio/gpiolib-legacy.o<br>
  CC      drivers/firmware/efi/runtime-wrappers.o<br>
  CC      lib/mpi/mpi-pow.o<br>
  CC      drivers/edac/cortex_arm64_edac.o<br>
  LD      kernel/sched/built-in.o<br>
  LD      drivers/firmware/qcom/built-in.o<br>
  CC      kernel/time/posix-timers.o<br>
  CC      fs/ext3/ext3_jbd.o<br>
  CC      net/core/dev_ioctl.o<br>
  CC      sound/soc/codecs/wsa881x.o<br>
  CC      lib/xz/xz_dec_syms.o<br>
  CC      net/core/tso.o<br>
  CC      drivers/gpio/gpiolib-of.o<br>
  LD      drivers/firmware/efi/built-in.o<br>
  LD      drivers/firmware/built-in.o<br>
  CC      kernel/time/posix-cpu-timers.o<br>
  CC      drivers/hid/hid-debug.o<br>
  CC      lib/xz/xz_dec_stream.o<br>
  CC      lib/mpi/mpiutil.o<br>
  CC      kernel/trace/trace_clock.o<br>
  CC      kernel/kcmp.o<br>
  LD      drivers/edac/edac_core.o<br>
  LD      drivers/edac/built-in.o<br>
  CC      kernel/trace/ftrace.o<br>
  CC      net/ipv6/udp.o<br>
  CC      drivers/gpio/gpiolib-sysfs.o<br>
  CC      lib/xz/xz_dec_lzma2.o<br>
  CC      kernel/trace/ring_buffer.o<br>
  CC      kernel/time/timekeeping.o<br>
  LD      lib/mpi/mpi.o<br>
  LD      lib/mpi/built-in.o<br>
  CC      kernel/time/ntp.o<br>
  CC      sound/soc/codecs/wsa881x-tables.o<br>
  CC      kernel/time/clocksource.o<br>
  CC      net/ipv4/tcp_input.o<br>
  CC      kernel/time/jiffies.o<br>
  CC      drivers/hid/hid-core.o<br>
  CC      net/core/flow.o<br>
  CC      sound/soc/codecs/wsa881x-regmap.o<br>
  CC      lib/xz/xz_dec_bcj.o<br>
  CC      sound/soc/codecs/wsa881x-analog.o<br>
  CC      drivers/gpio/qpnp-pin.o<br>
  CC      fs/ext3/xattr.o<br>
  CC      fs/ext3/xattr_user.o<br>
  CC      fs/ext3/xattr_trusted.o<br>
  LD      lib/xz/xz_dec.o<br>
  LD      lib/xz/built-in.o<br>
  CC      kernel/time/timer_list.o<br>
  CC      drivers/hwmon/hwmon.o<br>
  CC      lib/zlib_deflate/deflate.o<br>
  CC      drivers/gpu/msm/kgsl_trace.o<br>
  CC      net/core/net-sysfs.o<br>
  CC      fs/ext4/balloc.o<br>
  CC      lib/zlib_deflate/deftree.o<br>
  CC      lib/zlib_deflate/deflate_syms.o<br>
  CC      drivers/hid/hid-input.o<br>
  CC      kernel/time/timeconv.o<br>
  LD      fs/ext3/ext3.o<br>
  LD      fs/ext3/built-in.o<br>
  CC      drivers/hwmon/epm_adc.o<br>
  CC      drivers/gpio/gpio-msm-smp2p.o<br>
  CC      kernel/time/posix-clock.o<br>
  CC      kernel/time/alarmtimer.o<br>
  CC      kernel/time/clockevents.o<br>
  CC      sound/soc/codecs/wsa881x-tables-analog.o<br>
  CC      kernel/trace/trace.o<br>
  CC      net/ipv6/udplite.o<br>
  CC      kernel/trace/trace_output.o<br>
  LD      lib/zlib_deflate/zlib_deflate.o<br>
  LD      lib/zlib_deflate/built-in.o<br>
  CC      lib/zlib_inflate/inffast.o<br>
  CC      lib/textsearch.o<br>
  CC      sound/soc/codecs/wsa881x-regmap-analog.o<br>
  CC      lib/zlib_inflate/inflate.o<br>
  CC      drivers/hwmon/qpnp-adc-voltage.o<br>
  CC      fs/ext4/bitmap.o<br>
  CC      drivers/gpio/gpio-msm-smp2p-test.o<br>
  CC      lib/zlib_inflate/infutil.o<br>
  CC      net/ipv4/tcp_output.o<br>
  CC      kernel/time/tick-common.o<br>
  CC      net/ipv4/tcp_timer.o<br>
  CC      net/ipv6/raw.o<br>
  CC      sound/soc/codecs/wsa881x-irq.o<br>
  CC      lib/zlib_inflate/inftrees.o<br>
  CC      net/core/net-procfs.o<br>
  CC      fs/f2fs/dir.o<br>
  CC      fs/ext4/dir.o<br>
  LD      drivers/gpio/built-in.o<br>
  CC      lib/zlib_inflate/inflate_syms.o<br>
  CC      fs/f2fs/file.o<br>
  CC      net/ipv4/tcp_ipv4.o<br>
  CC      kernel/time/tick-broadcast.o<br>
  CC      drivers/hid/hidraw.o<br>
  CC      drivers/hid/uhid.o<br>
  CC      sound/soc/codecs/wsa881x-temp-sensor.o<br>
  LD      lib/zlib_inflate/zlib_inflate.o<br>
  LD      lib/zlib_inflate/built-in.o<br>
  CC      lib/ts_kmp.o<br>
  CC      drivers/gpu/msm/kgsl_cmdbatch.o<br>
  CC      net/core/fib_rules.o<br>
  CC      fs/ext4/file.o<br>
  CC      drivers/hwmon/qpnp-adc-common.o<br>
  CC      kernel/time/tick-broadcast-hrtimer.o<br>
  CC      fs/f2fs/inode.o<br>
  CC      lib/ts_bm.o<br>
  CC      drivers/hid/hid-generic.o<br>
  CC      kernel/time/sched_clock.o<br>
  CC      net/ipv6/icmp.o<br>
  CC      drivers/hwspinlock/hwspinlock_core.o<br>
  CC      sound/soc/codecs/msm_stub.o<br>
  CC      kernel/time/tick-oneshot.o<br>
  CC      fs/ext4/fsync.o<br>
  CC      lib/ts_fsm.o<br>
  CC      lib/percpu_counter.o<br>
  CC      kernel/time/tick-sched.o<br>
  CC      kernel/trace/trace_seq.o<br>
  CC      drivers/hid/hid-apple.o<br>
  LD      sound/soc/codecs/snd-soc-wcd9330.o<br>
  LD      sound/soc/codecs/snd-soc-wcd9335.o<br>
  CC      drivers/hwmon/qpnp-adc-current.o<br>
  CC      kernel/time/timekeeping_debug.o<br>
  LD      sound/soc/codecs/audio-ext-clock.o<br>
  LD      sound/soc/codecs/snd-soc-wcd9xxx.o<br>
  LD      sound/soc/codecs/snd-soc-wcd9xxx-v2.o<br>
  LD      sound/soc/codecs/snd-soc-msm8952-wcd.o<br>
  LD      sound/soc/codecs/snd-soc-wcd-cpe.o<br>
  CC      drivers/hwspinlock/msm_remote_spinlock.o<br>
  LD      sound/soc/codecs/snd-soc-wcd-mbhc.o<br>
  LD      sound/soc/codecs/snd-soc-wsa881x.o<br>
  LD      sound/soc/codecs/snd-soc-wsa881x-analog.o<br>
  CC      drivers/gpu/msm/kgsl_ioctl.o<br>
  LD      sound/soc/codecs/snd-soc-wsa881x-sensor.o<br>
  LD      sound/soc/codecs/snd-soc-msm-stub.o<br>
  LD      sound/soc/codecs/built-in.o<br>
  CC      drivers/gpu/msm/kgsl_sharedmem.o<br>
  CC      net/core/net-traces.o<br>
  CC      kernel/freezer.o<br>
  CC      fs/f2fs/namei.o<br>
  CC      lib/audit.o<br>
  CC      fs/ext4/ialloc.o<br>
  CC      drivers/hid/hid-elecom.o<br>
  CC      kernel/profile.o<br>
  CC      net/ipv4/tcp_minisocks.o<br>
  CC      kernel/trace/trace_stat.o<br>
  CC      sound/soc/msm/msm-pcm-hostless.o<br>
  CC      drivers/gpu/msm/kgsl_pwrctrl.o<br>
  CC      lib/compat_audit.o<br>
  LD      drivers/hwspinlock/built-in.o<br>
  BC      kernel/time/timeconst.h<br>
  CC      drivers/hid/hid-magicmouse.o<br>
  CC      drivers/hid/hid-microsoft.o<br>
  CC      lib/swiotlb.o<br>
  CC      kernel/time/time.o<br>
  CC      net/ipv6/mcast.o<br>
  CC      net/ipv6/reassembly.o<br>
  CC      kernel/trace/trace_printk.o<br>
  LD      drivers/hwmon/built-in.o<br>
  CC      drivers/i2c/i2c-boardinfo.o<br>
  CC      lib/iommu-helper.o<br>
  CC      drivers/gpu/msm/kgsl_pwrscale.o<br>
  CC      drivers/hid/hid-multitouch.o<br>
  CC      fs/f2fs/hash.o<br>
  CC      lib/syscall.o<br>
  CC      drivers/hid/usbhid/hid-core.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-dai-slim.o<br>
  CC      fs/ext4/inode.o<br>
  LD      kernel/time/built-in.o<br>
  CC      sound/soc/msm/qdsp6v2/audio_slimslave.o<br>
  CC      drivers/i2c/i2c-core.o<br>
  CC      net/ipv4/tcp_cong.o<br>
  CC      lib/nlattr.o<br>
  CC      kernel/trace/trace_sched_switch.o<br>
  CC      net/core/netclassid_cgroup.o<br>
  CC      fs/f2fs/super.o<br>
  CC      fs/f2fs/inline.o<br>
  LD      drivers/hid/hid.o<br>
  CC      net/core/sockev_nlmcast.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-dai-q6-v2.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-q6-v2.o<br>
  CC      net/ipv6/tcp_ipv6.o<br>
  CC      drivers/gpu/msm/kgsl_mmu.o<br>
  CC      kernel/trace/trace_functions.o<br>
  CC      drivers/hid/usbhid/hid-quirks.o<br>
  CC      drivers/gpu/msm/kgsl_snapshot.o<br>
  CC      kernel/trace/trace_cpu_freq_switch.o<br>
  CC      lib/checksum.o<br>
  CC      net/ipv6/ping.o<br>
  LD      net/core/built-in.o<br>
  CC      net/ipv4/tcp_metrics.o<br>
  CC      lib/cpu_rmap.o<br>
  CC      net/ipv4/tcp_fastopen.o<br>
  CC      fs/f2fs/checkpoint.o<br>
  CC      drivers/hid/usbhid/hiddev.o<br>
  CC      net/key/af_key.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-routing-v2.o<br>
  CC      fs/fat/cache.o<br>
  CC      kernel/trace/trace_nop.o<br>
  CC      lib/dynamic_queue_limits.o<br>
  CC      drivers/gpu/msm/kgsl_events.o<br>
  CC      fs/fat/dir.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-compress-q6-v2.o<br>
  CC      fs/ext4/page-io.o<br>
  CC      drivers/i2c/i2c-dev.o<br>
  CC      kernel/trace/trace_functions_graph.o<br>
  CC      lib/strncpy_from_user.o<br>
  CC      net/ipv6/exthdrs.o<br>
  LD      drivers/hid/usbhid/usbhid.o<br>
  LD      drivers/hid/usbhid/built-in.o<br>
  LD      drivers/hid/built-in.o<br>
  CC      net/ipv4/tcp_offload.o<br>
  CC      net/ipv4/datagram.o<br>
  CC      net/ipv4/raw.o<br>
  CC      lib/strnlen_user.o<br>
  CC      fs/fat/fatent.o<br>
  CC      drivers/i2c/i2c-mux.o<br>
  CC      fs/ext4/ioctl.o<br>
  CC      fs/fat/file.o<br>
  CC      drivers/gpu/msm/kgsl_pool.o<br>
  CC      lib/net_utils.o<br>
  CC      fs/f2fs/gc.o<br>
  CC      kernel/trace/blktrace.o<br>
  LD      net/key/built-in.o<br>
  CC      fs/fat/inode.o<br>
  CC      net/l2tp/l2tp_core.o<br>
  CC      net/l2tp/l2tp_ppp.o<br>
  CC      net/l2tp/l2tp_ip.o<br>
  CC      drivers/i2c/busses/i2c-msm-v2.o<br>
  CC      lib/asn1_decoder.o<br>
  CC      drivers/gpu/msm/kgsl_iommu.o<br>
  CC      drivers/gpu/msm/kgsl_debugfs.o<br>
  CC      fs/ext4/namei.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-lpa-v2.o<br>
  CC      fs/ext4/super.o<br>
  CC      net/ipv6/datagram.o<br>
  CC      net/ipv6/ip6_flowlabel.o<br>
  GEN     lib/oid_registry_data.c<br>
perl: warning: Setting locale failed.<br>
perl: warning: Please check that your locale settings:<br>
	LANGUAGE = (unset),<br>
	LC_ALL = (unset),<br>
	LC_CTYPE = "en_US.UTF-8",<br>
	LC_COLLATE = "C",<br>
	LC_MESSAGES = "en_US.UTF-8",<br>
	LC_NUMERIC = "C",<br>
	LANG = (unset)<br>
    are supported and installed on your system.<br>
perl: warning: Falling back to the standard locale ("C").<br>
  CC      kernel/trace/trace_events.o<br>
  CC      lib/ucs2_string.o<br>
  CC      net/ipv4/udp.o<br>
  CC      fs/f2fs/data.o<br>
  CC      fs/f2fs/node.o<br>
  CC      lib/qmi_encdec.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-afe-v2.o<br>
  CC      fs/fat/misc.o<br>
  LD      drivers/i2c/busses/built-in.o<br>
  CC      net/l2tp/l2tp_netlink.o<br>
  LD      drivers/i2c/built-in.o<br>
  CC      fs/f2fs/segment.o<br>
  CC      drivers/gpu/msm/kgsl_sync.o<br>
  CC      net/ipv4/udplite.o<br>
  CC      net/ipv6/inet6_connection_sock.o<br>
  CC      drivers/input/input.o<br>
  CC      lib/argv_split.o<br>
  CC      lib/bug.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-voip-v2.o<br>
  CC      kernel/trace/trace_export.o<br>
  CC      lib/clz_tab.o<br>
  CC      fs/fat/nfs.o<br>
  CC      lib/cmdline.o<br>
  CC      lib/cpumask.o<br>
  CC      fs/ext4/symlink.o<br>
  CC      fs/ext4/hash.o<br>
  CC      drivers/gpu/msm/kgsl_compat.o<br>
  CC      kernel/trace/trace_event_perf.o<br>
  CC      fs/fat/namei_vfat.o<br>
  CC      lib/ctype.o<br>
  CC      lib/dec_and_lock.o<br>
  CC      fs/ext4/resize.o<br>
  CC      net/l2tp/l2tp_eth.o<br>
  CC      net/l2tp/l2tp_debugfs.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-voice-v2.o<br>
  CC      lib/decompress.o<br>
  CC      net/ipv6/sysctl_net_ipv6.o<br>
  CC      net/ipv6/xfrm6_policy.o<br>
  CC      lib/decompress_bunzip2.o<br>
  CC      drivers/gpu/msm/adreno_ioctl.o<br>
  CC      drivers/input/input-compat.o<br>
  CC      drivers/input/serio/serio.o<br>
  CC      net/l2tp/l2tp_ip6.o<br>
  CC      kernel/trace/trace_events_filter.o<br>
  CC      net/ipv4/udp_offload.o<br>
  CC      fs/fat/namei_msdos.o<br>
  CC      drivers/gpu/msm/adreno_ringbuffer.o<br>
  CC      lib/decompress_inflate.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-dai-q6-hdmi-v2.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-lsm-client.o<br>
  CC      kernel/stacktrace.o<br>
  CC      kernel/futex.o<br>
  CC      drivers/input/input-mt.o<br>
  CC      net/ipv6/xfrm6_state.o<br>
  CC      lib/decompress_unlzma.o<br>
  CC      fs/f2fs/recovery.o<br>
  CC      fs/ext4/extents.o<br>
  CC      kernel/futex_compat.o<br>
  CC      drivers/input/serio/serport.o<br>
  LD      fs/fat/fat.o<br>
  LD      fs/fat/vfat.o<br>
  LD      fs/fat/msdos.o<br>
  LD      fs/fat/built-in.o<br>
  CC      fs/ext4/ext4_jbd2.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-host-voice-v2.o<br>
  LD      net/l2tp/built-in.o<br>
  CC      net/llc/llc_core.o<br>
  CC      drivers/gpu/msm/adreno_drawctxt.o<br>
  CC      drivers/input/ff-core.o<br>
  CC      net/ipv4/arp.o<br>
  CC      lib/dump_stack.o<br>
  CC      kernel/trace/trace_events_trigger.o<br>
  CC      net/ipv4/icmp.o<br>
  CC      net/ipv6/xfrm6_input.o<br>
  CC      drivers/input/serio/libps2.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-audio-effects-q6-v2.o<br>
  CC      fs/f2fs/shrinker.o<br>
  CC      lib/earlycpio.o<br>
  CC      fs/ext4/migrate.o<br>
  CC      kernel/smp.o<br>
  CC      lib/extable.o<br>
  CC      lib/fdt.o<br>
  CC      drivers/input/mousedev.o<br>
  CC      drivers/gpu/msm/adreno_dispatch.o<br>
  CC      drivers/input/evdev.o<br>
  CC      net/llc/llc_input.o<br>
  LD      drivers/input/serio/built-in.o<br>
  CC      lib/fdt_empty_tree.o<br>
  CC      drivers/input/fingerprint/fpc/fpc1020_tee.o<br>
  CC      fs/f2fs/extent_cache.o<br>
  CC      lib/fdt_ro.o<br>
  CC      kernel/trace/power-traces.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-loopback-v2.o<br>
  CC      net/ipv6/xfrm6_output.o<br>
  CC      net/ipv6/xfrm6_protocol.o<br>
  CC      net/ipv6/netfilter.o<br>
  CC      lib/fdt_rw.o<br>
  CC      lib/fdt_strerror.o<br>
  CC      lib/fdt_sw.o<br>
  LD      drivers/input/fingerprint/fpc/built-in.o<br>
  CC      drivers/input/fingerprint/goodix/gf_spi.o<br>
  CC      lib/fdt_wip.o<br>
  CC      net/ipv4/devinet.o<br>
  CC      drivers/input/fingerprint/goodix/platform.o<br>
  CC      drivers/input/fingerprint/goodix/netlink.o<br>
  CC      lib/flex_proportions.o<br>
  CC      fs/ext4/mballoc.o<br>
  CC      net/llc/llc_output.o<br>
  CC      sound/soc/msm/qdsp6v2/adsp_err.o<br>
  CC      drivers/input/joystick/xpad.o<br>
  CC      fs/f2fs/debug.o<br>
  CC      lib/idr.o<br>
  CC      kernel/trace/rpm-traces.o<br>
  CC      kernel/trace/ipc_logging.o<br>
  CC      drivers/gpu/msm/adreno_snapshot.o<br>
  CC      drivers/gpu/msm/adreno_coresight.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-dtmf-v2.o<br>
  CC      lib/int_sqrt.o<br>
  CC      lib/ioremap.o<br>
  CC      net/ipv6/fib6_rules.o<br>
  LD      drivers/input/fingerprint/goodix/built-in.o<br>
  LD      drivers/input/fingerprint/built-in.o<br>
  CC      drivers/gpu/msm/adreno_trace.o<br>
  CC      lib/irq_regs.o<br>
  LD      drivers/input/joystick/built-in.o<br>
  CC      drivers/input/keyboard/atkbd.o<br>
  LD      net/llc/llc.o<br>
  LD      net/llc/built-in.o<br>
  CC      drivers/input/keyboard/gpio_keys.o<br>
  CC      fs/f2fs/xattr.o<br>
  CC      drivers/input/misc/gpio_event.o<br>
  CC      fs/f2fs/acl.o<br>
  CC      lib/is_single_threaded.o<br>
  CC      drivers/input/touchscreen/of_touchscreen.o<br>
  CC      drivers/input/keyreset.o<br>
  CC      drivers/input/keycombo.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-dai-stub-v2.o<br>
  CC      lib/klist.o<br>
  CC      kernel/trace/ipc_logging_debug.o<br>
  CC      lib/kobject.o<br>
  CC      net/ipv6/proc.o<br>
  CC      drivers/input/misc/gpio_matrix.o<br>
  CC      net/ipv4/af_inet.o<br>
  CC      drivers/gpu/msm/adreno_a3xx.o<br>
  LD      fs/f2fs/f2fs.o<br>
  LD      drivers/input/keyboard/built-in.o<br>
  LD      fs/f2fs/built-in.o<br>
  CC      net/netlink/af_netlink.o<br>
  CC      net/packet/af_packet.o<br>
  CC      net/packet/diag.o<br>
  CC      drivers/input/touchscreen/ft5435/ft5435_ts.o<br>
  CC      net/rfkill/core.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-routing-devdep.o<br>
  CC      drivers/iommu/iommu.o<br>
  LD      kernel/trace/libftrace.o<br>
  LD      kernel/trace/built-in.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-dolby-dap-config.o<br>
  CC      kernel/uid16.o<br>
  CC      lib/kobject_uevent.o<br>
  CC      drivers/input/misc/gpio_input.o<br>
  CC      net/ipv6/ah6.o<br>
  CC      net/ipv6/esp6.o<br>
  CC      net/ipv6/ipcomp6.o<br>
  LD      net/rfkill/rfkill.o<br>
  LD      net/rfkill/built-in.o<br>
  CC      net/ipv6/xfrm6_tunnel.o<br>
  CC      kernel/system_keyring.o<br>
  CC      drivers/input/misc/gpio_output.o<br>
  CC      fs/ext4/block_validity.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-ds2-dap-config.o<br>
  CC      drivers/gpu/msm/adreno_a4xx.o<br>
  AS      kernel/system_certificates.o<br>
  CC      kernel/module.o<br>
  CC      lib/md5.o<br>
  CC      net/netfilter/core.o<br>
  CC      drivers/input/misc/gpio_axis.o<br>
  CC      drivers/iommu/iommu-traces.o<br>
  CC      drivers/iommu/iommu-sysfs.o<br>
  CC      lib/memcopy.o<br>
  CC      fs/ext4/move_extent.o<br>
  CC      sound/soc/msm/msm-dai-fe.o<br>
  CC      drivers/irqchip/irqchip.o<br>
  CC      net/ipv4/igmp.o<br>
  CC      lib/plist.o<br>
  CC      lib/proportions.o<br>
  CC      drivers/input/misc/hbtp_input.o<br>
  CC      lib/radix-tree.o<br>
  CC      net/netlink/genetlink.o<br>
  CC      net/ipv6/tunnel6.o<br>
  CC      drivers/gpu/msm/adreno_a5xx.o<br>
  CC      drivers/irqchip/irq-gic.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-dts-srs-tm-config.o<br>
  LD      drivers/input/touchscreen/ft5435/built-in.o<br>
  LD      drivers/input/touchscreen/gt9xx/built-in.o<br>
  CC      net/ipv6/xfrm6_mode_transport.o<br>
  CC      drivers/input/touchscreen/ist3038c/ist30xxc.o<br>
  CC      drivers/input/touchscreen/ist3038c/ist30xxc_misc.o<br>
  LD      net/packet/af_packet_diag.o<br>
  LD      net/packet/built-in.o<br>
  CC      fs/ext4/mmp.o<br>
  CC      fs/ext4/indirect.o<br>
  CC      drivers/iommu/msm_dma_iommu_mapping.o<br>
  CC      net/netfilter/nf_log.o<br>
  CC      drivers/input/misc/hbtp_vm.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-qti-pp-config.o<br>
  CC      drivers/irqchip/irq-gic-common.o<br>
  CC      lib/ratelimit.o<br>
  CC      drivers/iommu/io-pgtable.o<br>
  CC      sound/soc/msm/qdsp6v2/audio_calibration.o<br>
  CC      sound/soc/msm/qdsp6v2/audio_cal_utils.o<br>
  CC      net/ipv6/xfrm6_mode_tunnel.o<br>
  CC      lib/rbtree.o<br>
  CC      drivers/input/misc/keychord.o<br>
  CC      drivers/irqchip/irq-gic-v2m.o<br>
  CC      net/netlink/diag.o<br>
  CC      drivers/iommu/io-pgtable-arm.o<br>
  CC      drivers/iommu/io-pgtable-msm-secure.o<br>
  CC      kernel/kallsyms.o<br>
  CC      drivers/gpu/msm/adreno_a3xx_snapshot.o<br>
  CC      lib/sha1.o<br>
  CC      fs/ext4/extents_status.o<br>
  CC      net/netfilter/nf_queue.o<br>
  CC      lib/show_mem.o<br>
  CC      drivers/input/touchscreen/ist3038c/ist30xxc_sys.o<br>
  CC      lib/string.o<br>
  CC      drivers/irqchip/irq-gic-v3.o<br>
  CC      sound/soc/msm/qdsp6v2/q6adm.o<br>
  CC      drivers/input/misc/uinput.o<br>
  CC      net/ipv4/fib_frontend.o<br>
  CC      net/ipv6/xfrm6_mode_beet.o<br>
  CC      net/ipv4/fib_semantics.o<br>
  CC      drivers/iommu/of_iommu.o<br>
  CC      lib/timerqueue.o<br>
  CC      drivers/irqchip/irq-gic-v3-its.o<br>
  CC      drivers/gpu/msm/adreno_a4xx_snapshot.o<br>
  LD      net/netlink/netlink_diag.o<br>
  LD      net/netlink/built-in.o<br>
  CC      drivers/irqchip/irq-gic-v3-its-pci-msi.o<br>
  CC      kernel/compat.o<br>
  CC      lib/vsprintf.o<br>
  CC      sound/soc/msm/msm-cpe-lsm.o<br>
  CC      drivers/input/touchscreen/ist3038c/ist30xxc_update.o<br>
  CC      drivers/iommu/iommu-debug.o<br>
  LD      drivers/input/misc/built-in.o<br>
  LD      drivers/input/input-core.o<br>
  CC      sound/soc/msm/msm8952.o<br>
  CC      net/netfilter/nf_sockopt.o<br>
  CC      drivers/irqchip/irq-msm.o<br>
  CC      drivers/leds/led-core.o<br>
  CC      net/ipv6/mip6.o<br>
  CC      drivers/gpu/msm/adreno_a5xx_snapshot.o<br>
  CC      fs/ext4/xattr.o<br>
  CC      drivers/leds/led-class.o<br>
  CC      drivers/irqchip/msm_show_resume_irq.o<br>
  CC      net/ipv4/fib_trie.o<br>
  CC      net/ipv4/inet_fragment.o<br>
  CC      drivers/input/touchscreen/ist3038c/ist30xxc_tracking.o<br>
  LD      drivers/irqchip/built-in.o<br>
  CC      drivers/input/touchscreen/ist3038c/ist30xxc_cmcs.o<br>
  CC      drivers/gpu/msm/adreno_a4xx_preempt.o<br>
  CC      net/ipv6/netfilter/ip6_tables.o<br>
  CC      kernel/cgroup.o<br>
  CC      net/netfilter/nfnetlink.o<br>
  CC      drivers/gpu/msm/adreno_a5xx_preempt.o<br>
  CC      drivers/leds/led-triggers.o<br>
  CC      drivers/gpu/msm/adreno_sysfs.o<br>
  CC      drivers/gpu/msm/adreno.o<br>
  CC      sound/soc/msm/msm-audio-pinctrl.o<br>
  CC      sound/soc/msm/qdsp6v2/q6afe.o<br>
  CC      drivers/iommu/msm_iommu.o<br>
  CC      fs/ext4/xattr_user.o<br>
  GEN     lib/crc32table.h<br>
  CC      lib/oid_registry.o<br>
  CC      drivers/gpu/msm/adreno_cp_parser.o<br>
  CC      sound/soc/msm/msm8952-slimbus.o<br>
  CC      drivers/gpu/msm/adreno_perfcounter.o<br>
  CC      drivers/leds/leds-qpnp.o<br>
  CC      drivers/leds/leds-qpnp-flash.o<br>
  CC      net/ipv6/netfilter/ip6table_filter.o<br>
  CC      drivers/iommu/msm_iommu_domains.o<br>
  AR      lib/lib.a<br>
  CC      lib/crc32.o<br>
  CC      fs/ext4/xattr_trusted.o<br>
  CC      net/netfilter/nfnetlink_queue_core.o<br>
  LD      drivers/input/touchscreen/ist3038c/built-in.o<br>
  CC      drivers/input/touchscreen/synaptics_dsx/synaptics_dsx_i2c.o<br>
  CC      net/ipv4/ping.o<br>
  CC      drivers/gpu/msm/adreno_iommu.o<br>
  CC      drivers/gpu/msm/adreno_debugfs.o<br>
  CC      fs/ext4/inline.o<br>
  CC      net/ipv6/netfilter/ip6table_mangle.o<br>
  CC      fs/fuse/dev.o<br>
  CC      fs/jbd/transaction.o<br>
  LD      drivers/input/touchscreen/synaptics_dsx/built-in.o<br>
  CC      drivers/iommu/msm_iommu_mapping.o<br>
  CC      drivers/input/touchscreen/synaptics_dsx_2.6/synaptics_dsx_i2c.o<br>
  CC      fs/jbd/commit.o<br>
  LD      lib/built-in.o<br>
  CC      drivers/gpu/msm/adreno_profile.o<br>
  CC      drivers/leds/leds-qpnp-wled.o<br>
  CC      sound/soc/msm/qdsp6v2/q6asm.o<br>
  CC      drivers/leds/leds-aw2013.o<br>
  CC      net/netfilter/nfnetlink_log.o<br>
  CC      drivers/gpu/msm/adreno_compat.o<br>
  CC      net/ipv6/netfilter/ip6table_raw.o<br>
  CC      kernel/cgroup_freezer.o<br>
  CC      sound/soc/msm/qdsp6v2/q6audio-v2.o<br>
  CC      net/ipv4/ip_tunnel_core.o<br>
  LD      drivers/input/touchscreen/synaptics_dsx_2.6/built-in.o<br>
  CC      fs/ext4/xattr_security.o<br>
  CC      drivers/input/touchscreen/sweep2wake.o<br>
  CC      sound/soc/msm/msm8952-dai-links.o<br>
  CC      drivers/iommu/msm_iommu-v1.o<br>
  CC      fs/jbd/recovery.o<br>
  CC      drivers/input/touchscreen/sweep2sleep.o<br>
  LD      drivers/gpu/msm/msm_kgsl_core.o<br>
  LD      drivers/leds/built-in.o<br>
  LD      drivers/gpu/msm/msm_adreno.o<br>
  LD      drivers/gpu/msm/built-in.o<br>
  CC      net/netfilter/nf_conntrack_core.o<br>
  CC      kernel/cpuset.o<br>
  CC      drivers/iommu/msm_iommu_dev-v1.o<br>
  CC      drivers/gpu/vga/vgaarb.o<br>
  CC      kernel/utsname.o<br>
  CC      fs/fuse/dir.o<br>
  CC      net/ipv6/netfilter/nf_conntrack_l3proto_ipv6.o<br>
  LD      fs/ext4/ext4.o<br>
  LD      fs/ext4/built-in.o<br>
  CC      net/ipv4/gre_offload.o<br>
  CC      net/ipv4/ip_tunnel.o<br>
  LD      drivers/input/touchscreen/built-in.o<br>
  CC      kernel/user_namespace.o<br>
  CC      kernel/pid_namespace.o<br>
  CC      fs/jbd/checkpoint.o<br>
  LD      drivers/input/built-in.o<br>
  CC      fs/jbd/revoke.o<br>
  CC      net/netfilter/nf_conntrack_standalone.o<br>
  CC      drivers/iommu/msm_iommu_sec.o<br>
  CC      net/ipv4/sysctl_net_ipv4.o<br>
  LD      drivers/gpu/vga/built-in.o<br>
  CC      net/ipv6/netfilter/nf_conntrack_proto_icmpv6.o<br>
  LD      drivers/gpu/built-in.o<br>
  CC      net/ipv6/netfilter/nf_nat_l3proto_ipv6.o<br>
  CC      net/ipv6/sit.o<br>
  CC      drivers/iommu/msm_iommu_pagetable.o<br>
  GZIP    kernel/config_data.gz<br>
  CC      fs/fuse/file.o<br>
  CC      drivers/iommu/arm-smmu.o<br>
  CC      kernel/res_counter.o<br>
  CC      fs/jbd/journal.o<br>
  LD      sound/soc/msm/snd-soc-hostless-pcm.o<br>
  CC      net/ipv4/sysfs_net_ipv4.o<br>
  CC      kernel/stop_machine.o<br>
../../../../../../kernel/xiaomi/msm8953/net/netfilter/nf_conntrack_core.c: In function 'nf_conntrack_init_net':<br>
../../../../../../kernel/xiaomi/msm8953/net/netfilter/nf_conntrack_core.c:1787:20: warning: unused variable 'unique_id' [-Wunused-variable]<br>
  static atomic64_t unique_id;<br>
                    ^<br>
../../../../../../kernel/xiaomi/msm8953/net/netfilter/nf_conntrack_core.c: At top level:<br>
cc1: warning: unrecognized command line option "-Wno-misleading-indentation"<br>
  CC      kernel/audit.o<br>
  CC      kernel/auditfilter.o<br>
  CC      net/netfilter/nf_conntrack_expect.o<br>
  CC      net/ipv4/proc.o<br>
  CC      net/ipv4/fib_rules.o<br>
  CC      fs/fuse/inode.o<br>
  CC      net/ipv4/udp_tunnel.o<br>
  LD      sound/soc/msm/snd-soc-qdsp6v2.o<br>
  CC      fs/jbd2/transaction.o<br>
  CC      net/ipv6/netfilter/nf_nat_proto_icmpv6.o<br>
  CC      sound/soc/msm/qdsp6v2/q6voice.o<br>
  CC      fs/jbd2/commit.o<br>
  CC      net/ipv6/netfilter/nf_defrag_ipv6_hooks.o<br>
  CC      net/ipv6/addrconf_core.o<br>
  CC      net/ipv4/ah4.o<br>
  CC      net/ipv6/netfilter/nf_conntrack_reasm.o<br>
  LD      drivers/iommu/built-in.o<br>
  CC      kernel/auditsc.o<br>
  LD      fs/jbd/jbd.o<br>
  CC      net/netfilter/nf_conntrack_helper.o<br>
  CC      net/ipv4/esp4.o<br>
  LD      fs/jbd/built-in.o<br>
  CC      net/ipv6/netfilter/nf_log_ipv6.o<br>
  CC      fs/fuse/control.o<br>
  CC      fs/kernfs/mount.o<br>
  CC      drivers/md/dm-uevent.o<br>
  CC      fs/jbd2/recovery.o<br>
  CC      net/ipv6/netfilter/nf_reject_ipv6.o<br>
  CC      kernel/audit_watch.o<br>
  CC      fs/jbd2/checkpoint.o<br>
  CC      fs/kernfs/inode.o<br>
  CC      drivers/media/i2c/ir-kbd-i2c.o<br>
  CC      fs/fuse/shortcircuit.o<br>
  CC      net/ipv6/exthdrs_core.o<br>
  CC      net/ipv6/netfilter/ip6t_rpfilter.o<br>
  CC      drivers/md/dm.o<br>
  CC      drivers/md/dm-table.o<br>
  CC      drivers/md/dm-target.o<br>
  CC      fs/jbd2/revoke.o<br>
  CC      net/ipv4/ipcomp.o<br>
  CC      fs/kernfs/dir.o<br>
  LD      fs/fuse/fuse.o<br>
  LD      fs/fuse/built-in.o<br>
  CC      net/netfilter/nf_conntrack_proto.o<br>
  CC      net/netfilter/nf_conntrack_l3proto_generic.o<br>
  CC      fs/nls/nls_base.o<br>
  LD      drivers/media/i2c/built-in.o<br>
  CC      fs/nls/nls_cp437.o<br>
  CC      fs/nls/nls_cp936.o<br>
  CC      kernel/audit_tree.o<br>
  CC      net/ipv6/netfilter/ip6t_REJECT.o<br>
  CC      fs/nls/nls_cp950.o<br>
  CC      drivers/md/dm-linear.o<br>
  CC      fs/jbd2/journal.o<br>
  CC      kernel/seccomp.o<br>
  CC      kernel/utsname_sysctl.o<br>
  CC      drivers/media/platform/msm/camera_v2/camera/camera.o<br>
  CC      net/ipv4/xfrm4_tunnel.o<br>
  CC      fs/kernfs/file.o<br>
  CC      fs/nls/nls_ascii.o<br>
  CC      drivers/media/platform/msm/camera_v2/common/msm_camera_io_util.o<br>
  CC      net/netfilter/nf_conntrack_proto_generic.o<br>
  CC      drivers/media/platform/msm/camera_v2/common/cam_smmu_api.o<br>
  CC      sound/soc/msm/qdsp6v2/q6core.o<br>
  LD      sound/soc/msm/snd-soc-cpe.o<br>
  CC      sound/soc/msm/qdsp6v2/rtac.o<br>
  CC      sound/soc/msm/qdsp6v2/q6lsm.o<br>
  LD      net/ipv6/netfilter/nf_conntrack_ipv6.o<br>
  LD      net/ipv6/netfilter/nf_nat_ipv6.o<br>
  LD      sound/soc/msm/snd-soc-msm8x16.o<br>
  LD      net/ipv6/netfilter/nf_defrag_ipv6.o<br>
  CC      kernel/tracepoint.o<br>
  LD      net/ipv6/netfilter/built-in.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_dev.o<br>
  CC      net/ipv6/ip6_checksum.o<br>
  CC      fs/nls/nls_iso8859-1.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_core.o<br>
  CC      drivers/md/dm-stripe.o<br>
  CC      net/ipv4/tunnel4.o<br>
  LD      drivers/media/platform/msm/camera_v2/camera/built-in.o<br>
  CC      fs/kernfs/symlink.o<br>
  CC      fs/nls/nls_utf8.o<br>
  CC      net/netfilter/nf_conntrack_proto_tcp.o<br>
  CC      drivers/media/platform/msm/camera_v2/fd/msm_fd_dev.o<br>
  CC      drivers/media/platform/msm/camera_v2/fd/msm_fd_hw.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-q6-noirq.o<br>
  CC      kernel/elfcore.o<br>
  CC      kernel/irq_work.o<br>
  CC      drivers/md/dm-ioctl.o<br>
  LD      fs/nls/built-in.o<br>
  LD      fs/kernfs/built-in.o<br>
  CC      kernel/cpu_pm.o<br>
  LD      sound/soc/snd-soc-core.o<br>
  CC      drivers/md/dm-io.o<br>
  CC      drivers/md/dm-kcopyd.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_base.o<br>
  CC      drivers/media/platform/msm/camera_v2/common/cam_hw_ops.o<br>
  CC      net/ipv6/ip6_icmp.o<br>
  LD      sound/soc/msm/qdsp6v2/snd-soc-qdsp6v2.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_formats.o<br>
  CC      net/ipv6/output_core.o<br>
  CC      net/ipv4/xfrm4_mode_transport.o<br>
  LD      sound/soc/msm/qdsp6v2/built-in.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_util.o<br>
  LD      sound/soc/msm/built-in.o<br>
  LD      drivers/media/platform/msm/camera_v2/fd/built-in.o<br>
  CHK     kernel/config_data.h<br>
  UPD     kernel/config_data.h<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_io_util.o<br>
  CC      kernel/configs.o<br>
  LD      fs/jbd2/jbd2.o<br>
  LD      sound/soc/built-in.o<br>
  LD      fs/jbd2/built-in.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_smmu.o<br>
  LD      sound/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/common/cam_soc_api.o<br>
  CC      fs/notify/fsnotify.o<br>
  CC      drivers/md/dm-sysfs.o<br>
  CC      drivers/md/dm-stats.o<br>
  CC      net/netfilter/nf_conntrack_proto_udp.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1_wb.o<br>
  LD      kernel/built-in.o<br>
  CC      net/rmnet_data/rmnet_data_main.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1_pipe.o<br>
  CC      drivers/mfd/mfd-core.o<br>
  CC      drivers/misc/uid_stat.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1_ctl.o<br>
  CC      net/ipv4/xfrm4_mode_tunnel.o<br>
  CC      net/sched/sch_generic.o<br>
  CC      net/sched/sch_mq.o<br>
  CC      drivers/md/dm-builtin.o<br>
  LD      fs/overlayfs/built-in.o<br>
  CC      net/ipv6/protocol.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1.o<br>
  CC      fs/notify/notification.o<br>
  CC      fs/notify/group.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r3.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_sync.o<br>
  CC      net/ipv4/ipconfig.o<br>
  LD      drivers/media/platform/msm/camera_v2/common/built-in.o<br>
  CC      drivers/mfd/wcd9xxx-core.o<br>
  CC      net/netfilter/nf_conntrack_extend.o<br>
  CC      drivers/md/dm-bufio.o<br>
  CC      drivers/md/dm-crypt.o<br>
  CC      net/netfilter/nf_conntrack_acct.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_buf_mgr.o<br>
  CC      net/netfilter/nf_conntrack_seqadj.o<br>
  CC      net/rmnet_data/rmnet_data_config.o<br>
  CC      fs/notify/inode_mark.o<br>
  CC      drivers/media/platform/msm/vidc/msm_v4l2_vidc.o<br>
  CC      drivers/misc/qcom/qdsp6v2/aac_in.o<br>
  CC      net/rmnet_data/rmnet_data_vnd.o<br>
  CC      net/ipv6/ip6_offload.o<br>
  CC      fs/notify/mark.o<br>
  CC      net/sched/sch_api.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_debug.o<br>
  CC      net/netfilter/nf_conntrack_ecache.o<br>
  CC      net/netfilter/nf_conntrack_proto_dccp.o<br>
  CC      drivers/misc/qcom/qdsp6v2/qcelp_in.o<br>
  CC      net/ipv4/netfilter.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp_util.o<br>
  CC      drivers/media/platform/msm/vidc/msm_vidc_common.o<br>
  CC      fs/notify/vfsmount_mark.o<br>
  CC      drivers/media/platform/msm/vidc/msm_vidc.o<br>
  CC      net/sched/sch_blackhole.o<br>
  CC      fs/notify/fdinfo.o<br>
  CC      drivers/md/dm-verity-fec.o<br>
  CC      net/ipv6/tcpv6_offload.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1_debug.o<br>
  CC      net/rmnet_data/rmnet_data_handlers.o<br>
  CC      drivers/mfd/wcd9xxx-irq.o<br>
  CC      drivers/misc/qcom/qdsp6v2/evrc_in.o<br>
  CC      drivers/mfd/wcd9xxx-slimslave.o<br>
  CC      fs/notify/dnotify/dnotify.o<br>
  CC      net/sched/cls_api.o<br>
  CC      net/netfilter/nf_conntrack_proto_gre.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r3_debug.o<br>
  CC      fs/notify/fanotify/fanotify.o<br>
  CC      drivers/md/dm-verity-target.o<br>
  CC      net/ipv6/udp_offload.o<br>
  CC      drivers/misc/qcom/qdsp6v2/amrnb_in.o<br>
  LD      fs/notify/dnotify/built-in.o<br>
  CC      drivers/misc/qcom/qdsp6v2/g711mlaw_in.o<br>
  CC      net/sched/act_api.o<br>
  CC      drivers/mfd/wcd9xxx-core-resource.o<br>
  CC      fs/notify/inotify/inotify_fsnotify.o<br>
  CC      net/ipv4/netfilter/nf_conntrack_l3proto_ipv4_compat.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp_axi_util.o<br>
  CC      net/rmnet_data/rmnet_map_data.o<br>
  CC      net/rmnet_data/rmnet_map_command.o<br>
  CC      fs/notify/fanotify/fanotify_user.o<br>
  LD      drivers/media/platform/msm/sde/rotator/built-in.o<br>
  LD      drivers/media/platform/msm/sde/built-in.o<br>
  CC      net/rmnet_data/rmnet_data_stats.o<br>
  CC      drivers/mfd/wcd9335-regmap.o<br>
  CC      drivers/misc/qcom/qdsp6v2/g711alaw_in.o<br>
  CC      fs/notify/inotify/inotify_user.o<br>
  CC      net/ipv6/exthdrs_offload.o<br>
  CC      net/netfilter/nf_conntrack_proto_sctp.o<br>
  CC      drivers/mfd/wcd9335-tables.o<br>
  CC      drivers/mfd/wcd-gpio-ctrl.o<br>
  CC      drivers/md/dm-req-crypt.o<br>
  CC      drivers/misc/qseecom.o<br>
  CC      net/ipv4/netfilter/nf_conntrack_l3proto_ipv4.o<br>
  CC      drivers/misc/hdcp.o<br>
  LD      fs/notify/fanotify/built-in.o<br>
  CC      drivers/media/platform/msm/vidc/msm_vdec.o<br>
  CC      drivers/media/platform/msm/vidc/msm_venc.o<br>
  CC      drivers/media/platform/msm/camera_v2/ispif/msm_ispif.o<br>
  LD      net/rmnet_data/rmnet_data.o<br>
  LD      net/rmnet_data/built-in.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_utils.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_wma.o<br>
  LD      fs/notify/inotify/built-in.o<br>
  LD      fs/notify/built-in.o<br>
  LD      drivers/mfd/built-in.o<br>
  CC      net/netfilter/nf_conntrack_proto_udplite.o<br>
  CC      net/ipv6/inet6_hashtables.o<br>
  CC      net/sched/sch_fifo.o<br>
  CC      net/ipv6/ip6_udp_tunnel.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp_stats_util.o<br>
  CC      fs/proc/task_mmu.o<br>
  CC      drivers/misc/compat_qseecom.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_wmapro.o<br>
  CC      net/ipv4/netfilter/nf_conntrack_proto_icmp.o<br>
  LD      drivers/md/dm-mod.o<br>
  LD      drivers/md/dm-verity.o<br>
  CC      fs/proc/inode.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp48.o<br>
  LD      drivers/md/built-in.o<br>
  CC      drivers/mmc/card/block.o<br>
  CC      net/netfilter/nf_conntrack_netlink.o<br>
  LD      drivers/media/platform/msm/camera_v2/ispif/built-in.o<br>
  CC      drivers/mmc/card/queue.o<br>
  CC      drivers/misc/type-c-pericom.o<br>
  CC      net/sched/sch_htb.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_aac.o<br>
  LD      net/ipv6/ipv6.o<br>
  CC      net/unix/af_unix.o<br>
  CC      drivers/mmc/core/core.o<br>
  LD      net/ipv6/built-in.o<br>
  CC      fs/proc/root.o<br>
  CC      net/wireguard/main.o<br>
  CC      net/unix/garbage.o<br>
  CC      net/ipv4/netfilter/nf_nat_l3proto_ipv4.o<br>
  CC      drivers/misc/fastchg.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp47.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp46.o<br>
  CC      fs/proc/base.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_multi_aac.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_alac.o<br>
  CC      drivers/media/platform/msm/vidc/msm_smem.o<br>
  CC      net/wireguard/noise.o<br>
  CC      net/unix/sysctl_net_unix.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp44.o<br>
  CC      net/sched/sch_prio.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_ape.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_utils_aio.o<br>
  CC      net/ipv4/netfilter/nf_nat_proto_icmp.o<br>
  CC      net/netfilter/nf_conntrack_amanda.o<br>
  CC      net/netfilter/nf_conntrack_ftp.o<br>
  CC      drivers/media/platform/msm/vidc/msm_vidc_debug.o<br>
  CC      drivers/media/platform/msm/vidc/msm_vidc_res_parse.o<br>
  CC      net/unix/diag.o<br>
  CC      drivers/mmc/core/bus.o<br>
  CC      drivers/media/platform/msm/vidc/venus_hfi.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp40.o<br>
  CC      net/ipv4/netfilter/nf_defrag_ipv4.o<br>
  CC      net/wireguard/device.o<br>
  CC      net/sched/cls_u32.o<br>
  LD      drivers/mmc/card/mmc_block.o<br>
  CC      fs/proc/generic.o<br>
  CC      net/wireguard/peer.o<br>
  LD      drivers/mmc/card/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp.o<br>
  CC      drivers/misc/qcom/qdsp6v2/q6audio_v2.o<br>
  CC      net/netfilter/nf_conntrack_h323_main.o<br>
  CC      net/wireless/core.o<br>
  CC      drivers/mmc/core/host.o<br>
  LD      net/unix/unix.o<br>
  LD      net/unix/unix_diag.o<br>
  LD      net/unix/built-in.o<br>
  CC      net/wireless/sysfs.o<br>
  CC      drivers/media/platform/msm/vidc/hfi_response_handler.o<br>
  CC      net/netfilter/nf_conntrack_h323_asn1.o<br>
  CC      drivers/misc/qcom/qdsp6v2/q6audio_v2_aio.o<br>
  CC      fs/proc/array.o<br>
  CC      net/wireguard/timers.o<br>
  CC      net/sched/cls_fw.o<br>
  CC      net/wireguard/queueing.o<br>
  CC      net/wireguard/send.o<br>
  CC      net/ipv4/netfilter/nf_log_ipv4.o<br>
  CC      drivers/media/platform/msm/vidc/hfi_packetization.o<br>
  CC      net/netfilter/nf_conntrack_irc.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_mp3.o<br>
  CC      drivers/mmc/core/mmc.o<br>
  LD      drivers/media/platform/msm/camera_v2/isp/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_dev.o<br>
  CC      fs/proc/fd.o<br>
  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_sync.o<br>
  CC      net/sched/cls_flow.o<br>
  CC      net/sched/cls_cgroup.o<br>
  CC      net/sched/ematch.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_amrnb.o<br>
  CC      drivers/media/platform/msm/camera_v2/msm_buf_mgr/msm_generic_buf_mgr.o<br>
  CC      net/ipv4/netfilter/nf_reject_ipv4.o<br>
  CC      net/wireless/radiotap.o<br>
  CC      fs/proc/proc_tty.o<br>
  CC      net/wireguard/receive.o<br>
  CC      net/netfilter/nf_conntrack_broadcast.o<br>
  CC      drivers/media/radio/radio-iris.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_amrwb.o<br>
  CC      drivers/media/radio/radio-iris-transport.o<br>
  CC      net/xfrm/xfrm_policy.o<br>
  CC      net/netfilter/nf_conntrack_netbios_ns.o<br>
  CC      drivers/media/platform/msm/vidc/vidc_hfi.o<br>
  CC      fs/proc/cmdline.o<br>
  CC      fs/proc/consoles.o<br>
  CC      net/sched/em_cmp.o<br>
  LD      drivers/media/platform/msm/camera_v2/msm_buf_mgr/built-in.o<br>
  CC      fs/proc/cpuinfo.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_amrwbplus.o<br>
  CC      drivers/mmc/core/mmc_ops.o<br>
  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_core.o<br>
  CC      net/wireless/util.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_evrc.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_qcelp.o<br>
  CC      net/ipv4/netfilter/nf_nat_h323.o<br>
  CC      drivers/media/platform/msm/vidc/venus_boot.o<br>
  CC      net/wireguard/socket.o<br>
  CC      fs/proc/devices.o<br>
  CC      fs/proc/interrupts.o<br>
  CC      net/netfilter/nf_conntrack_pptp.o<br>
  CC      net/netfilter/nf_conntrack_sane.o<br>
  CC      drivers/misc/qcom/qdsp6v2/amrwb_in.o<br>
  CC      net/sched/em_nbyte.o<br>
  CC      net/sched/em_u32.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_hwacc_effects.o<br>
  CC      fs/proc/loadavg.o<br>
  CC      fs/proc/meminfo.o<br>
  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_hw.o<br>
  CC      drivers/mmc/core/sd.o<br>
  CC      drivers/media/platform/msm/vidc/msm_vidc_dcvs.o<br>
  CC      fs/proc/stat.o<br>
  CC      drivers/media/radio/silabs/radio-silabs.o<br>
  CC      net/ipv4/netfilter/nf_nat_pptp.o<br>
  CC      fs/proc/uptime.o<br>
  CC      net/netfilter/nf_conntrack_tftp.o<br>
  CC      net/sched/em_meta.o<br>
  CC      net/sched/em_text.o<br>
  CC      net/wireguard/peerlookup.o<br>
  CC      drivers/misc/qcom/qdsp6v2/ultrasound/usf.o<br>
  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_platform.o<br>
  CC      net/wireguard/allowedips.o<br>
  CC      net/netfilter/nf_log_common.o<br>
  CC      net/netfilter/nf_nat_core.o<br>
  CC      fs/proc/version.o<br>
  CC      drivers/mmc/core/sd_ops.o<br>
  CC      drivers/media/platform/msm/vidc/governors/msm_vidc_dyn_gov.o<br>
  CC      fs/proc/softirqs.o<br>
  CC      net/ipv4/inet_diag.o<br>
  CC      net/ipv4/netfilter/nf_nat_masquerade_ipv4.o<br>
  CC      net/xfrm/xfrm_state.o<br>
  CC      net/wireless/reg.o<br>
  LD      drivers/media/platform/msm/camera_v2/jpeg_10/built-in.o<br>
  CC      drivers/media/platform/msm/vidc/governors/msm_vidc_table_gov.o<br>
  CC      drivers/mmc/core/sdio.o<br>
  CC      drivers/media/platform/msm/camera_v2/msm_vb2/msm_vb2.o<br>
  LD      net/sched/built-in.o<br>
  CC      net/compat.o<br>
  CC      net/sysctl_net.o<br>
  CC      fs/proc/namespaces.o<br>
  CC      drivers/misc/qcom/qdsp6v2/ultrasound/usfcdev.o<br>
  CC      net/wireguard/ratelimiter.o<br>
  LD      drivers/media/radio/silabs/built-in.o<br>
  LD      drivers/media/radio/built-in.o<br>
  CC      drivers/misc/qcom/qdsp6v2/ultrasound/q6usm.o<br>
  CC      net/netfilter/nf_nat_proto_unknown.o<br>
  CC      net/netfilter/nf_nat_proto_common.o<br>
  CC      fs/proc/self.o<br>
  CC      net/netfilter/nf_nat_proto_udp.o<br>
  LD      drivers/media/platform/msm/vidc/governors/built-in.o<br>
  CC      net/ipv4/netfilter/nf_nat_proto_gre.o<br>
  CC      net/activity_stats.o<br>
  CC      drivers/media/platform/msm/vidc/vmem/vmem.o<br>
  CC      drivers/mmc/core/sdio_ops.o<br>
  LD      drivers/media/platform/msm/camera_v2/msm_vb2/built-in.o<br>
  CC      drivers/media/platform/msm/vidc/vmem/vmem_debugfs.o<br>
  CC      net/netfilter/nf_nat_proto_tcp.o<br>
  CC      drivers/media/platform/msm/camera_v2/pproc/cpp/msm_cpp_soc.o<br>
  CC      fs/proc/thread_self.o<br>
  CC      drivers/media/platform/msm/camera_v2/pproc/cpp/msm_cpp.o<br>
  CC      net/wireguard/cookie.o<br>
  LD      drivers/misc/qcom/qdsp6v2/ultrasound/built-in.o<br>
  LD      drivers/misc/qcom/qdsp6v2/built-in.o<br>
  CC      net/wireguard/netlink.o<br>
  LD      drivers/misc/qcom/built-in.o<br>
  CC      net/wireguard/crypto/zinc/chacha20/chacha20.o<br>
  LD      drivers/misc/built-in.o<br>
  CC      drivers/mmc/core/sdio_bus.o<br>
  CC      drivers/mmc/core/sdio_cis.o<br>
  CC      fs/proc/proc_sysctl.o<br>
  CC      fs/proc/proc_net.o<br>
  LD      drivers/media/platform/msm/vidc/vmem/built-in.o<br>
  LD      drivers/media/platform/msm/vidc/built-in.o<br>
  CC      net/xfrm/xfrm_hash.o<br>
  CC      net/xfrm/xfrm_input.o<br>
  CC      net/wireless/scan.o<br>
  CC      drivers/mmc/core/sdio_io.o<br>
  CC      net/ipv4/netfilter/ip_tables.o<br>
  CC      net/ipv4/netfilter/iptable_filter.o<br>
  CC      net/netfilter/nf_nat_helper.o<br>
  CC      net/netfilter/nf_nat_proto_dccp.o<br>
  CC      net/xfrm/xfrm_output.o<br>
  CC      drivers/mmc/core/sdio_irq.o<br>
  CC      drivers/mmc/core/quirks.o<br>
  CC      net/netfilter/nf_nat_proto_udplite.o<br>
  CC      net/netfilter/nf_nat_proto_sctp.o<br>
  CC      drivers/mmc/core/slot-gpio.o<br>
  CC      net/netfilter/nf_nat_amanda.o<br>
  CC      net/netfilter/nf_nat_ftp.o<br>
  CC      fs/proc/kmsg.o<br>
  CC      fs/proc/page.o<br>
  CC      net/xfrm/xfrm_sysctl.o<br>
  CC      net/ipv4/tcp_diag.o<br>
  CC      net/netfilter/nf_nat_irc.o<br>
  PERLASM net/wireguard/crypto/zinc/chacha20/chacha20-arm64.S<br>
perl: warning: Setting locale failed.<br>
perl: warning: Please check that your locale settings:<br>
	LANGUAGE = (unset),<br>
	LC_ALL = (unset),<br>
	LC_CTYPE = "en_US.UTF-8",<br>
	LC_COLLATE = "C",<br>
	LC_MESSAGES = "en_US.UTF-8",<br>
	LC_NUMERIC = "C",<br>
	LANG = (unset)<br>
    are supported and installed on your system.<br>
perl: warning: Falling back to the standard locale ("C").<br>
  CC      net/xfrm/xfrm_replay.o<br>
  CC      net/xfrm/xfrm_proc.o<br>
  CC      net/wireguard/crypto/zinc/poly1305/poly1305.o<br>
  CC      net/xfrm/xfrm_algo.o<br>
  CC      drivers/mmc/core/debugfs.o<br>
  LD      fs/proc/proc.o<br>
  CC      net/netfilter/nf_nat_tftp.o<br>
  LD      fs/proc/built-in.o<br>
  CC      net/netfilter/x_tables.o<br>
  CC      drivers/net/macvlan.o<br>
  LD      drivers/media/platform/msm/camera_v2/pproc/cpp/built-in.o<br>
  LD      drivers/media/platform/msm/camera_v2/pproc/built-in.o<br>
  CC      net/ipv4/netfilter/iptable_mangle.o<br>
  CC      net/wireless/nl80211.o<br>
  CC      fs/pstore/inode.o<br>
  CC      drivers/mmc/host/sdhci.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/actuator/msm_actuator.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/cci/msm_cci.o<br>
  CC      drivers/mmc/host/sdhci-pltfm.o<br>
  CC      drivers/mmc/host/sdhci-msm.o<br>
  LD      drivers/mmc/core/mmc_core.o<br>
  LD      drivers/mmc/core/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/csid/msm_csid.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/csiphy/msm_csiphy.o<br>
  CC      fs/pstore/platform.o<br>
  CC      fs/pstore/ftrace.o<br>
  CC      net/xfrm/xfrm_user.o<br>
  PERLASM net/wireguard/crypto/zinc/poly1305/poly1305-arm64.S<br>
  CC      net/ipv4/netfilter/iptable_nat.o<br>
perl: warning: Setting locale failed.<br>
perl: warning: Please check that your locale settings:<br>
	LANGUAGE = (unset),<br>
	LC_ALL = (unset),<br>
	LC_CTYPE = "en_US.UTF-8",<br>
	LC_COLLATE = "C",<br>
	LC_MESSAGES = "en_US.UTF-8",<br>
	LC_NUMERIC = "C",<br>
	LANG = (unset)<br>
    are supported and installed on your system.<br>
perl: warning: Falling back to the standard locale ("C").<br>
  CC      net/wireguard/crypto/zinc/chacha20poly1305.o<br>
  CC      drivers/media/platform/msm/camera_v2/msm.o<br>
  CC      net/wireless/mlme.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/actuator/built-in.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/cci/built-in.o<br>
  CC      drivers/media/rc/rc-main.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/eeprom/msm_eeprom.o<br>
  CC      fs/pstore/ram.o<br>
  CC      drivers/media/rc/keymaps/rc-adstech-dvb-t-pci.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/csid/built-in.o<br>
  CC      net/netfilter/xt_tcpudp.o<br>
  CC      drivers/media/rc/keymaps/rc-alink-dtu-m.o<br>
  CC      net/ipv4/netfilter/iptable_raw.o<br>
  CC      drivers/net/mii.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/csiphy/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-anysee.o<br>
  CC      drivers/media/rc/keymaps/rc-apac-viewcomp.o<br>
  CC      drivers/media/rc/keymaps/rc-asus-pc39.o<br>
  CC      fs/pstore/ram_core.o<br>
  CC      net/wireless/ibss.o<br>
  CC      net/ipv4/netfilter/iptable_security.o<br>
  CC      net/wireless/sme.o<br>
  CC      drivers/nfc/nq-nci.o<br>
  CC      drivers/media/rc/keymaps/rc-asus-ps3-100.o<br>
  CC      net/netfilter/xt_mark.o<br>
  CC      net/wireguard/crypto/zinc/blake2s/blake2s.o<br>
  CC      net/netfilter/xt_connmark.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/eeprom/built-in.o<br>
  CC      net/xfrm/xfrm_ipcomp.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/flash/msm_flash.o<br>
  CC      drivers/net/Space.o<br>
  LD      fs/pstore/pstore.o<br>
  LD      fs/pstore/ramoops.o<br>
  LD      fs/pstore/built-in.o<br>
  CC      drivers/mmc/host/sdhci-msm-ice.o<br>
  CC      fs/quota/dquot.o<br>
  CC      drivers/media/tuners/tuner-xc2028.o<br>
  CC      drivers/media/rc/keymaps/rc-ati-tv-wonder-hd-600.o<br>
  CC      drivers/media/rc/keymaps/rc-ati-x10.o<br>
  CC      drivers/media/rc/keymaps/rc-avermedia-a16d.o<br>
  CC      net/ipv4/netfilter/ipt_ah.o<br>
  LD      drivers/nfc/built-in.o<br>
  CC      net/netfilter/xt_nat.o<br>
  CC      drivers/net/loopback.o<br>
  CC      drivers/mmc/host/cmdq_hci.o<br>
  CC      drivers/media/rc/rc-ir-raw.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/flash/built-in.o<br>
  CC      drivers/of/base.o<br>
  CC      drivers/media/rc/keymaps/rc-avermedia.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/ir_cut/msm_ir_cut.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/ir_led/msm_ir_led.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_cci_i2c.o<br>
  CC      net/ipv4/netfilter/ipt_MASQUERADE.o<br>
  LD      net/xfrm/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_qup_i2c.o<br>
  CC      net/netfilter/xt_CHECKSUM.o<br>
  CC      drivers/media/rc/keymaps/rc-avermedia-cardbus.o<br>
  CC      drivers/media/rc/lirc_dev.o<br>
  CC      drivers/media/rc/ir-nec-decoder.o<br>
  CC      drivers/media/tuners/tuner-simple.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/ir_led/built-in.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/ir_cut/built-in.o<br>
  CC      fs/quota/quota.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/ois/msm_ois.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_spi.o<br>
  CC      drivers/net/phy/phy.o<br>
  CC      drivers/media/rc/keymaps/rc-avermedia-dvbt.o<br>
  LD      drivers/mmc/host/built-in.o<br>
  CC      net/ipv4/netfilter/ipt_NATTYPE.o<br>
  LD      drivers/mmc/built-in.o<br>
  CC      net/netfilter/xt_CLASSIFY.o<br>
  CC      net/netfilter/xt_CONNSECMARK.o<br>
  CC      net/ipv4/netfilter/ipt_REJECT.o<br>
  CC      drivers/of/device.o<br>
  CC      drivers/net/ppp/ppp_generic.o<br>
  CC      net/wireguard/crypto/zinc/curve25519/curve25519.o<br>
  CC      drivers/net/ethernet/msm/rndis_ipa.o<br>
  CC      drivers/media/rc/keymaps/rc-avermedia-m135a.o<br>
  CC      drivers/net/ppp/ppp_async.o<br>
  CC      fs/quota/kqid.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_dt_util.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/ois/built-in.o<br>
  CC      net/wireless/chan.o<br>
  CC      drivers/media/tuners/tuner-types.o<br>
  CC      net/wireless/ethtool.o<br>
  CC      net/netfilter/xt_CT.o<br>
  CC      drivers/media/rc/keymaps/rc-avermedia-m733a-rm-k6.o<br>
  CC      drivers/of/platform.o<br>
  CC      drivers/of/fdt.o<br>
  CC      net/ipv4/netfilter/arp_tables.o<br>
  CC      drivers/net/phy/phy_device.o<br>
  LD      fs/quota/built-in.o<br>
  CC      fs/ramfs/inode.o<br>
  CC      drivers/media/rc/keymaps/rc-avermedia-rm-ks.o<br>
  CC      drivers/media/tuners/mt20xx.o<br>
  CC      drivers/media/tuners/tda8290.o<br>
  CC      drivers/of/fdt_address.o<br>
  CC      net/netfilter/xt_LOG.o<br>
  CC      fs/ramfs/file-mmu.o<br>
  CC      drivers/media/rc/keymaps/rc-avertv-303.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/io/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/msm_sensor_init.o<br>
  CC      net/wireguard/compat/siphash/siphash.o<br>
  CC      net/wireguard/compat/dst_cache/dst_cache.o<br>
  LD      drivers/net/ethernet/msm/built-in.o<br>
  CC      net/wireless/mesh.o<br>
  CC      drivers/of/address.o<br>
  LD      fs/ramfs/ramfs.o<br>
  LD      fs/ramfs/built-in.o<br>
  CC      fs/squashfs/block.o<br>
  CC      drivers/net/ppp/bsd_comp.o<br>
  CC      drivers/media/rc/keymaps/rc-azurewave-ad-tu700.o<br>
  CC      drivers/media/rc/ir-rc5-decoder.o<br>
  CC      net/ipv4/tcp_cubic.o<br>
  CC      drivers/net/phy/mdio_bus.o<br>
  CC      net/netfilter/xt_NETMAP.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/msm_sensor_driver.o<br>
  CC      net/ipv4/netfilter/arpt_mangle.o<br>
  LD      drivers/net/ethernet/built-in.o<br>
  CC      drivers/media/tuners/tea5767.o<br>
  CC      drivers/media/rc/ir-rc6-decoder.o<br>
  CC      drivers/media/rc/keymaps/rc-behold.o<br>
  CC      fs/squashfs/cache.o<br>
  CC      drivers/media/rc/ir-jvc-decoder.o<br>
  CC      drivers/net/ppp/ppp_deflate.o<br>
  CC      drivers/net/ppp/ppp_mppe.o<br>
  CC      drivers/of/irq.o<br>
  AS      net/wireguard/crypto/zinc/chacha20/chacha20-arm64.o<br>
  CC      drivers/media/rc/keymaps/rc-behold-columbus.o<br>
  AS      net/wireguard/crypto/zinc/poly1305/poly1305-arm64.o<br>
  CC      drivers/media/rc/keymaps/rc-budget-ci-old.o<br>
  LD      net/wireguard/wireguard.o<br>
  CC      drivers/media/rc/ir-sony-decoder.o<br>
  LD      net/wireguard/built-in.o<br>
  CC      drivers/media/tuners/tea5761.o<br>
  CC      fs/squashfs/dir.o<br>
  CC      net/netfilter/xt_NFLOG.o<br>
  CC      drivers/net/slip/slhc.o<br>
  CC      net/ipv4/netfilter/arptable_filter.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/msm_sensor.o<br>
  LD      drivers/net/phy/libphy.o<br>
  LD      drivers/net/phy/built-in.o<br>
  CC      net/wireless/ap.o<br>
  CC      drivers/media/tuners/tda9887.o<br>
  LD      drivers/net/wireless/ath/built-in.o<br>
  CC      drivers/net/ppp/ppp_synctty.o<br>
  CC      drivers/media/tuners/tda827x.o<br>
  CC      drivers/net/usb/asix_devices.o<br>
  CC      drivers/pci/access.o<br>
  CC      drivers/media/rc/keymaps/rc-cinergy-1400.o<br>
  LD      drivers/net/wireless/ath/wil6210/built-in.o<br>
  CC      fs/squashfs/export.o<br>
  CC      drivers/net/wireless/cnss_crypto/cnss_secif.o<br>
  CC      drivers/pci/bus.o<br>
  CC      drivers/of/of_net.o<br>
  CC      net/netfilter/xt_NFQUEUE.o<br>
  CC      drivers/media/tuners/tda18271-maps.o<br>
  CC      drivers/media/rc/keymaps/rc-cinergy.o<br>
  LD      net/ipv4/netfilter/nf_conntrack_ipv4.o<br>
  LD      net/ipv4/netfilter/nf_nat_ipv4.o<br>
  LD      net/ipv4/netfilter/built-in.o<br>
  CC      net/ipv4/tcp_highspeed.o<br>
  CC      fs/squashfs/file.o<br>
  LD      drivers/net/wireless/cnss_crypto/built-in.o<br>
  CC      drivers/net/wireless/cnss_prealloc/cnss_prealloc.o<br>
  CC      drivers/net/ppp/pppox.o<br>
  CC      drivers/net/usb/asix_common.o<br>
  CC      net/wireless/trace.o<br>
  CC      drivers/media/rc/keymaps/rc-delock-61959.o<br>
  CC      drivers/net/wireless/wcnss/wcnss_wlan.o<br>
  CC      drivers/media/rc/keymaps/rc-dib0700-nec.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-dib0700-rc5.o<br>
  LD      drivers/media/platform/msm/camera_v2/built-in.o<br>
  CC      drivers/pci/probe.o<br>
  LD      drivers/media/platform/msm/built-in.o<br>
  CC      drivers/media/tuners/tda18271-common.o<br>
  CC      drivers/of/of_mdio.o<br>
  LD      drivers/net/slip/built-in.o<br>
  CC      drivers/of/of_pci.o<br>
  CC      fs/squashfs/fragment.o<br>
  CC      drivers/media/platform/soc_camera/soc_camera.o<br>
  CC      net/netfilter/xt_REDIRECT.o<br>
  CC      drivers/media/platform/soc_camera/soc_mediabus.o<br>
  CC      drivers/net/tun.o<br>
  LD      drivers/net/wireless/cnss_prealloc/built-in.o<br>
  CC      drivers/media/platform/soc_camera/soc_camera_platform.o<br>
  CC      net/ipv4/tcp_htcp.o<br>
  CC      drivers/media/rc/keymaps/rc-digitalnow-tinytwin.o<br>
  CC      fs/squashfs/id.o<br>
  CC      drivers/media/tuners/tda18271-fe.o<br>
  CC      drivers/net/ppp/pppoe.o<br>
  CC      drivers/media/rc/keymaps/rc-digittrade.o<br>
  CC      drivers/net/usb/ax88172a.o<br>
  CC      drivers/of/of_pci_irq.o<br>
  CC      fs/sysfs/file.o<br>
  CC      fs/squashfs/inode.o<br>
  CC      fs/sysfs/dir.o<br>
  CC      net/netfilter/xt_SECMARK.o<br>
  CC      fs/tracefs/inode.o<br>
  CC      drivers/media/rc/keymaps/rc-dm1105-nec.o<br>
  CC      drivers/pci/host-bridge.o<br>
  CC      net/ipv4/tcp_vegas.o<br>
  CC      drivers/media/rc/keymaps/rc-dntv-live-dvb-t.o<br>
  CC      fs/squashfs/namei.o<br>
  CC      drivers/of/of_spmi.o<br>
  CC      drivers/net/usb/ax88179_178a.o<br>
  CC      drivers/net/usb/cdc_ether.o<br>
  CC      fs/sysfs/symlink.o<br>
  LD      drivers/media/platform/soc_camera/built-in.o<br>
  LD      drivers/media/platform/built-in.o<br>
  CC      net/netfilter/xt_TPROXY.o<br>
  LD      fs/tracefs/tracefs.o<br>
  LD      fs/tracefs/built-in.o<br>
  CC      drivers/net/usb/net1080.o<br>
  CC      drivers/media/rc/keymaps/rc-dntv-live-dvbt-pro.o<br>
  CC      drivers/net/wireless/wcnss/wcnss_vreg.o<br>
  CC      drivers/pci/remove.o<br>
  CC      fs/squashfs/super.o<br>
  CC      drivers/net/ppp/pppolac.o<br>
  CC      drivers/media/tuners/xc5000.o<br>
  CC      drivers/media/tuners/xc4000.o<br>
  CC      drivers/of/of_reserved_mem.o<br>
  CC      fs/sysfs/mount.o<br>
  CC      drivers/media/rc/keymaps/rc-dvbsky.o<br>
  CC      net/ipv4/tcp_veno.o<br>
  CC      fs/squashfs/symlink.o<br>
  CC      drivers/pci/pci.o<br>
  CC      fs/squashfs/decompressor.o<br>
  LD      drivers/net/wireless/wcnss/wcnsscore.o<br>
  CC      drivers/media/v4l2-core/v4l2-dev.o<br>
  LD      drivers/net/wireless/wcnss/built-in.o<br>
  LD      drivers/net/wireless/built-in.o<br>
  CC      drivers/media/media-device.o<br>
  CC      drivers/net/veth.o<br>
  CC      net/netfilter/xt_TCPMSS.o<br>
  CC      fs/sysfs/group.o<br>
  CC      drivers/media/rc/keymaps/rc-em-terratec.o<br>
  CC      drivers/net/usb/cdc_subset.o<br>
  CC      drivers/net/ppp/pppopns.o<br>
  CC      drivers/of/of_slimbus.o<br>
  CC      fs/squashfs/file_cache.o<br>
  CC      fs/squashfs/decompressor_single.o<br>
  CC      net/ipv4/tcp_scalable.o<br>
  CC      drivers/media/rc/keymaps/rc-encore-enltv2.o<br>
  CC      drivers/media/rc/keymaps/rc-encore-enltv.o<br>
  LD      fs/sysfs/built-in.o<br>
  CC      drivers/pci/pci-driver.o<br>
  CC      drivers/of/of_coresight.o<br>
  CC      drivers/pci/search.o<br>
  CC      drivers/media/v4l2-core/v4l2-ioctl.o<br>
  CC      fs/squashfs/xz_wrapper.o<br>
  CC      drivers/media/tuners/mc44s803.o<br>
  CC      drivers/net/usb/zaurus.o<br>
  CC      net/netfilter/xt_TEE.o<br>
  CC      drivers/media/v4l2-core/v4l2-device.o<br>
  CC      drivers/media/rc/keymaps/rc-encore-enltv-fm53.o<br>
  CC      drivers/media/media-devnode.o<br>
  CC      drivers/media/rc/ir-sanyo-decoder.o<br>
  LD      drivers/net/ppp/built-in.o<br>
  CC      drivers/media/rc/ir-sharp-decoder.o<br>
  CC      net/ipv4/tcp_lp.o<br>
  CC      drivers/of/of_batterydata.o<br>
  CC      fs/squashfs/zlib_wrapper.o<br>
  CC      drivers/media/rc/keymaps/rc-evga-indtube.o<br>
  CC      drivers/media/rc/ir-lirc-codec.o<br>
  CC      drivers/media/rc/ir-xmp-decoder.o<br>
  CC      drivers/media/rc/pwm-ir.o<br>
  LD      drivers/media/tuners/tda18271.o<br>
  CC      drivers/net/usb/usbnet.o<br>
  LD      drivers/media/tuners/built-in.o<br>
  CC      drivers/media/media-entity.o<br>
  CC      drivers/net/usb/cdc_ncm.o<br>
  CC      drivers/media/v4l2-core/v4l2-fh.o<br>
  CC      drivers/pci/pci-sysfs.o<br>
  LD      fs/squashfs/squashfs.o<br>
  CC      drivers/media/rc/keymaps/rc-eztv.o<br>
  LD      fs/squashfs/built-in.o<br>
  LD      drivers/of/built-in.o<br>
  CC      net/netfilter/xt_TRACE.o<br>
  CC      fs/eventpoll.o<br>
  CC      drivers/media/v4l2-core/v4l2-event.o<br>
  LD      drivers/media/rc/rc-core.o<br>
  CC      net/wireless/wext-core.o<br>
  CC      drivers/phy/phy-core.o<br>
  CC      drivers/phy/phy-qcom-ufs.o<br>
  CC      net/ipv4/tcp_yeah.o<br>
  CC      drivers/media/rc/keymaps/rc-flydvb.o<br>
  CC      drivers/pinctrl/core.o<br>
  CC      net/ipv4/tcp_illinois.o<br>
  CC      drivers/media/v4l2-core/v4l2-ctrls.o<br>
  CC      net/netfilter/xt_IDLETIMER.o<br>
  CC      drivers/pci/rom.o<br>
  CC      drivers/media/rc/keymaps/rc-flyvideo.o<br>
  CC      drivers/media/rc/keymaps/rc-fusionhdtv-mce.o<br>
  CC      net/netfilter/xt_HARDIDLETIMER.o<br>
  CC      drivers/media/v4l2-core/v4l2-subdev.o<br>
  CC      drivers/phy/phy-qcom-ufs-qmp-20nm.o<br>
  CC      fs/anon_inodes.o<br>
  CC      drivers/phy/phy-qcom-ufs-qmp-14nm.o<br>
  CC      drivers/phy/phy-qcom-ufs-qmp-v3.o<br>
  LD      drivers/net/usb/asix.o<br>
  CC      net/wireless/wext-proc.o<br>
  CC      drivers/media/rc/keymaps/rc-gadmei-rm008z.o<br>
  CC      net/ipv4/tcp_memcontrol.o<br>
  CC      drivers/phy/phy-qcom-ufs-qrbtc-v2.o<br>
  CC      drivers/pci/setup-res.o<br>
  CC      net/ipv4/xfrm4_policy.o<br>
  CC      net/netfilter/xt_comment.o<br>
  CC      net/wireless/wext-spy.o<br>
  CC      drivers/pinctrl/pinctrl-utils.o<br>
  CC      fs/signalfd.o<br>
  LD      drivers/net/usb/built-in.o<br>
  LD      drivers/net/built-in.o<br>
  CC      net/wireless/wext-priv.o<br>
  CC      net/netfilter/xt_connlimit.o<br>
  CC      net/netfilter/xt_conntrack.o<br>
  CC      drivers/media/rc/keymaps/rc-genius-tvgo-a11mce.o<br>
  CC      net/netfilter/xt_dscp.o<br>
  LD      drivers/phy/built-in.o<br>
  CC      fs/timerfd.o<br>
  CC      drivers/platform/msm/ipa/ipa_clients/odu_bridge.o<br>
  CC      drivers/platform/msm/ipa/ipa_clients/ipa_mhi_client.o<br>
  CC      drivers/pci/irq.o<br>
  CC      drivers/pinctrl/pinmux.o<br>
  CC      drivers/media/rc/keymaps/rc-gotview7135.o<br>
  CC      net/netfilter/xt_ecn.o<br>
  CC      drivers/platform/msm/msm_11ad/msm_11ad.o<br>
  CC      net/ipv4/xfrm4_state.o<br>
  CC      net/ipv4/xfrm4_input.o<br>
  CC      net/wireless/regdb.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_core.o<br>
  CC      drivers/media/rc/keymaps/rc-imon-mce.o<br>
  LD      drivers/media/media.o<br>
  CC      drivers/media/v4l2-core/v4l2-clk.o<br>
  CC      fs/eventfd.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_client_api.o<br>
  CC      drivers/pinctrl/pinconf.o<br>
  CC      drivers/pci/vpd.o<br>
  CC      drivers/pinctrl/devicetree.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa.o<br>
  CC      drivers/media/rc/keymaps/rc-imon-pad.o<br>
  CC      net/netfilter/xt_esp.o<br>
  CC      net/netfilter/xt_hashlimit.o<br>
  CC      net/netfilter/xt_helper.o<br>
  CC      drivers/pci/setup-bus.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_of.o<br>
  CC      drivers/pinctrl/pinconf-generic.o<br>
  CC      drivers/power/power_supply_core.o<br>
  LD      net/wireless/cfg80211.o<br>
  CC      drivers/media/v4l2-core/v4l2-async.o<br>
  LD      net/wireless/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-iodata-bctv7e.o<br>
  CC      fs/aio.o<br>
  CC      drivers/power/power_supply_sysfs.o<br>
  CC      drivers/power/power_supply_leds.o<br>
  CC      drivers/platform/msm/ipa/ipa_clients/ipa_uc_offload.o<br>
  CC      net/ipv4/xfrm4_output.o<br>
  LD      drivers/platform/msm/msm_11ad/msm_11ad_proxy.o<br>
  LD      drivers/platform/msm/msm_11ad/built-in.o<br>
  CC      net/netfilter/xt_hl.o<br>
  CC      drivers/pinctrl/qcom/pinctrl-msm.o<br>
  CC      drivers/media/rc/keymaps/rc-it913x-v1.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_debugfs.o<br>
  CC      drivers/pinctrl/qcom/pinctrl-msm8953.o<br>
  CC      drivers/power/smb1351-charger.o<br>
  CC      drivers/platform/msm/spmi/spmi.o<br>
  CC      net/netfilter/xt_iprange.o<br>
  CC      drivers/media/v4l2-core/v4l2-compat-ioctl32.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_rpm_smd.o<br>
  CC      net/netfilter/xt_l2tp.o<br>
  CC      drivers/media/rc/keymaps/rc-it913x-v2.o<br>
  CC      drivers/platform/msm/spmi/spmi-resources.o<br>
  CC      drivers/pci/vc.o<br>
  CC      drivers/pci/proc.o<br>
  LD      drivers/platform/msm/ipa/ipa_clients/built-in.o<br>
  CC      net/ipv4/xfrm4_protocol.o<br>
  CC      drivers/platform/msm/sps/bam.o<br>
  CC      drivers/media/rc/keymaps/rc-kaiomy.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_fabric_adhoc.o<br>
  LD      drivers/pinctrl/qcom/built-in.o<br>
  LD      drivers/pinctrl/built-in.o<br>
  CC      drivers/platform/msm/sps/sps_bam.o<br>
  CC      drivers/platform/msm/sps/sps.o<br>
  CC      drivers/platform/msm/spmi/spmi-pmic-arb.o<br>
  CC      drivers/platform/msm/ipa/ipa_api.o<br>
  CC      drivers/pwm/core.o<br>
  CC      fs/locks.o<br>
  CC      drivers/pci/slot.o<br>
  CC      drivers/media/rc/keymaps/rc-kworld-315u.o<br>
  CC      net/netfilter/xt_length.o<br>
  CC      drivers/media/rc/keymaps/rc-kworld-pc150u.o<br>
  CC      drivers/media/rc/keymaps/rc-kworld-plus-tv-analog.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_arb_adhoc.o<br>
  CC      drivers/power/smb135x-charger.o<br>
  CC      drivers/media/rc/keymaps/rc-leadtek-y04g0051.o<br>
  LD      net/ipv4/built-in.o<br>
  CC      drivers/pci/quirks.o<br>
  CC      drivers/platform/msm/spmi/qpnp-int.o<br>
  CC      drivers/pwm/sysfs.o<br>
  CC      drivers/media/rc/keymaps/rc-lirc.o<br>
  CC      drivers/media/v4l2-core/v4l2-of.o<br>
  CC      net/netfilter/xt_limit.o<br>
  CC      fs/compat.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_hdr.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_flt.o<br>
  CC      drivers/media/rc/keymaps/rc-lme2510.o<br>
  CC      drivers/pwm/pwm-qpnp.o<br>
  CC      drivers/media/v4l2-core/v4l2-common.o<br>
  CC      drivers/media/rc/keymaps/rc-manli.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_rules.o<br>
  CC      net/netfilter/xt_mac.o<br>
  CC      net/netfilter/xt_multiport.o<br>
  CC      drivers/platform/msm/spmi/spmi-dbgfs.o<br>
  CC      drivers/platform/msm/ipa/ipa_rm.o<br>
  CC      drivers/media/rc/keymaps/rc-medion-x10.o<br>
  CC      drivers/media/rc/keymaps/rc-medion-x10-digitainer.o<br>
  CC      drivers/media/rc/keymaps/rc-medion-x10-or2x.o<br>
  CC      fs/compat_ioctl.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_rt.o<br>
  CC      drivers/power/qpnp-fg.o<br>
  CC      drivers/media/v4l2-core/v4l2-dv-timings.o<br>
  CC      net/netfilter/xt_pkttype.o<br>
  LD      drivers/platform/msm/spmi/built-in.o<br>
  CC      net/netfilter/xt_policy.o<br>
  CC      drivers/pci/msi.o<br>
  CC      drivers/pci/setup-irq.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_dp.o<br>
  CC      drivers/pci/pci-label.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_bimc_adhoc.o<br>
  CC      drivers/pci/syscall.o<br>
  LD      drivers/pwm/built-in.o<br>
  CC      drivers/platform/msm/ipa/ipa_rm_dependency_graph.o<br>
  CC      drivers/media/rc/keymaps/rc-msi-digivox-ii.o<br>
  CC      drivers/platform/msm/ipa/ipa_rm_peers_list.o<br>
  CC      drivers/pci/of.o<br>
  CC      drivers/media/v4l2-core/v4l2-mem2mem.o<br>
  CC      drivers/media/rc/keymaps/rc-msi-digivox-iii.o<br>
  CC      drivers/media/rc/keymaps/rc-msi-tvanywhere.o<br>
  CC      drivers/media/v4l2-core/videobuf-core.o<br>
  CC      drivers/media/v4l2-core/videobuf2-core.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_noc_adhoc.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_of_adhoc.o<br>
  CC      net/netfilter/xt_qtaguid_print.o<br>
  CC      drivers/platform/msm/ipa/ipa_rm_resource.o<br>
  CC      drivers/media/rc/keymaps/rc-msi-tvanywhere-plus.o<br>
  CC      fs/binfmt_script.o<br>
  CC      drivers/pci/host/pci-msm.o<br>
  CC      drivers/ras/ras.o<br>
  CC      fs/binfmt_elf.o<br>
  CC      net/netfilter/xt_qtaguid.o<br>
  CC      fs/compat_binfmt_elf.o<br>
  CC      drivers/platform/msm/msm_bus/msm_buspm_coresight_adhoc.o<br>
  CC      drivers/media/rc/keymaps/rc-nebula.o<br>
  CC      net/netfilter/xt_quota.o<br>
  CC      drivers/media/rc/keymaps/rc-nec-terratec-cinergy-xs.o<br>
  CC      fs/mbcache.o<br>
  CC      drivers/media/v4l2-core/videobuf2-memops.o<br>
  CC      drivers/platform/msm/ipa/ipa_rm_inactivity_timer.o<br>
  CC      drivers/media/rc/keymaps/rc-norwood.o<br>
  CC      drivers/media/rc/keymaps/rc-npgtech.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_dbg.o<br>
  CC      drivers/ras/debugfs.o<br>
  CC      net/netfilter/xt_quota2.o<br>
  CC      drivers/media/rc/keymaps/rc-pctv-sedna.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_client.o<br>
  CC      fs/posix_acl.o<br>
  CC      drivers/media/v4l2-core/videobuf2-vmalloc.o<br>
  CC      drivers/platform/msm/qpnp-power-on.o<br>
  CC      drivers/platform/msm/qpnp-revid.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_utils.o<br>
  LD      drivers/media/v4l2-core/videodev.o<br>
  CC      drivers/platform/msm/sps/sps_dma.o<br>
  CC      net/netfilter/xt_socket.o<br>
  LD      drivers/ras/built-in.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_nat.o<br>
  CC      drivers/power/qpnp-smbcharger.o<br>
  CC      drivers/media/rc/keymaps/rc-pinnacle-color.o<br>
  CC      drivers/platform/msm/sps/sps_map.o<br>
  CC      drivers/platform/msm/qpnp-coincell.o<br>
  CC      drivers/media/rc/keymaps/rc-pinnacle-grey.o<br>
  LD      drivers/media/v4l2-core/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-pinnacle-pctv-hd.o<br>
  CC      drivers/platform/msm/qpnp-haptic.o<br>
  CC      fs/coredump.o<br>
  CC      fs/drop_caches.o<br>
  CC      drivers/platform/msm/sps/sps_mem.o<br>
  CC      drivers/platform/msm/sps/sps_rm.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_intf.o<br>
../../../../../../kernel/xiaomi/msm8953/drivers/power/qpnp-fg.c:2590:12: warning: 'read_beat' defined but not used [-Wunused-function]<br>
 static int read_beat(struct fg_chip *chip, u8 *beat_count)<br>
            ^<br>
cc1: warning: unrecognized command line option "-Wno-misleading-indentation"<br>
  CC      drivers/power/pmic-voter.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/teth_bridge.o<br>
  LD      drivers/platform/msm/msm_bus/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-pixelview.o<br>
  CC      drivers/media/rc/keymaps/rc-pixelview-mk12.o<br>
  CC      drivers/platform/msm/usb_bam.o<br>
  CC      drivers/media/rc/keymaps/rc-pixelview-002t.o<br>
  CC      net/netfilter/xt_state.o<br>
  CC      drivers/regulator/core.o<br>
  CC      drivers/regulator/dummy.o<br>
  CC      drivers/media/rc/keymaps/rc-pixelview-new.o<br>
  CC      drivers/regulator/fixed-helper.o<br>
  CC      drivers/media/rc/keymaps/rc-powercolor-real-angel.o<br>
  CC      drivers/rtc/rtc-lib.o<br>
  LD      drivers/platform/msm/sps/built-in.o<br>
  CC      drivers/platform/msm/avtimer.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_interrupts.o<br>
  CC      drivers/media/rc/keymaps/rc-proteus-2309.o<br>
  CC      drivers/power/qpnp-typec.o<br>
  CC      fs/fhandle.o<br>
  CC      drivers/sensors/sensors_ssc.o<br>
  CC      fs/dcookies.o<br>
  CC      net/netfilter/xt_statistic.o<br>
  CC      drivers/scsi/scsi.o<br>
  CC      drivers/rtc/hctosys.o<br>
  CC      drivers/slimbus/slimbus.o<br>
  CC      drivers/media/rc/keymaps/rc-purpletv.o<br>
  CC      drivers/slimbus/slim-msm.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_uc.o<br>
  CC      drivers/rtc/systohc.o<br>
  LD      drivers/sensors/built-in.o<br>
  LD      drivers/platform/msm/ipa/ipa_common<br>
  CC      drivers/slimbus/slim-msm-ngd.o<br>
  CC      drivers/power/qns_system.o<br>
  LD      fs/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-pv951.o<br>
  CC      net/netfilter/xt_string.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_uc_wdi.o<br>
  CC      drivers/rtc/class.o<br>
  AS      drivers/soc/qcom/idle-v8.o<br>
  CC      drivers/soc/qcom/cpu_ops.o<br>
../../../../../../kernel/xiaomi/msm8953/drivers/power/qpnp-smbcharger.c: In function 'aicl_done_handler':<br>
../../../../../../kernel/xiaomi/msm8953/drivers/power/qpnp-smbcharger.c:6662:6: warning: unused variable 'aicl_level' [-Wunused-variable]<br>
  int aicl_level = smbchg_get_aicl_level_ma(chip);<br>
      ^<br>
../../../../../../kernel/xiaomi/msm8953/drivers/power/qpnp-smbcharger.c: At top level:<br>
cc1: warning: unrecognized command line option "-Wno-misleading-indentation"<br>
  CC      drivers/power/battery_current_limit.o<br>
  CC      drivers/media/rc/keymaps/rc-hauppauge.o<br>
  CC      drivers/media/rc/keymaps/rc-rc6-mce.o<br>
  CC      drivers/power/msm_bcl.o<br>
  CC      drivers/power/bcl_peripheral.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_dma.o<br>
  CC      drivers/soc/qcom/msm_rq_stats.o<br>
  CC      net/netfilter/xt_time.o<br>
  CC      net/netfilter/xt_u32.o<br>
  CC      drivers/media/rc/keymaps/rc-real-audio-220-32-keys.o<br>
  CC      drivers/rtc/interface.o<br>
  CC      drivers/scsi/hosts.o<br>
  CC      drivers/regulator/helpers.o<br>
  CC      drivers/scsi/scsi_ioctl.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_uc_mhi.o<br>
  LD      drivers/slimbus/built-in.o<br>
  CC      drivers/soundwire/soundwire.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_mhi.o<br>
  CC      drivers/media/rc/keymaps/rc-reddo.o<br>
  CC      drivers/soc/qcom/cpuss_dump.o<br>
  CC      drivers/media/rc/keymaps/rc-snapstream-firefly.o<br>
  CC      drivers/media/rc/keymaps/rc-streamzap.o<br>
  LD      net/netfilter/netfilter.o<br>
  LD      net/netfilter/nfnetlink_queue.o<br>
  LD      net/netfilter/nf_conntrack.o<br>
  LD      net/netfilter/nf_conntrack_h323.o<br>
  LD      net/netfilter/nf_nat.o<br>
  CC      drivers/power/qcom/msm-pm.o<br>
  LD      net/netfilter/built-in.o<br>
  CC      drivers/regulator/devres.o<br>
  CC      drivers/regulator/of_regulator.o<br>
  LD      net/built-in.o<br>
  CC      drivers/rtc/rtc-dev.o<br>
  CC      drivers/soc/qcom/memory_dump_v2.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_uc_ntn.o<br>
  CC      drivers/scsi/constants.o<br>
  CC      drivers/media/rc/keymaps/rc-tbs-nec.o<br>
  LD      drivers/power/power_supply.o<br>
  CC      drivers/power/reset/msm-poweroff.o<br>
  CC      drivers/regulator/fixed.o<br>
  CC      drivers/soundwire/swr-wcd-ctrl.o<br>
  CC      drivers/regulator/mem-acc-regulator.o<br>
  CC      drivers/power/qcom/pm-data.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/rmnet_ipa.o<br>
  LD      drivers/pci/host/built-in.o<br>
  LD      drivers/pci/built-in.o<br>
  CC      drivers/spi/spi.o<br>
  CC      drivers/media/rc/keymaps/rc-technisat-usb2.o<br>
  CC      drivers/media/rc/keymaps/rc-terratec-cinergy-xs.o<br>
  CC      drivers/media/rc/keymaps/rc-terratec-slim.o<br>
  CC      drivers/rtc/rtc-proc.o<br>
  CC      drivers/soc/qcom/ddr-health.o<br>
  CC      drivers/media/rc/keymaps/rc-terratec-slim-2.o<br>
  CC      drivers/media/rc/keymaps/rc-tevii-nec.o<br>
  LD      drivers/power/reset/built-in.o<br>
  CC      drivers/power/qcom/lpm-stats.o<br>
  CC      drivers/regulator/fan53555.o<br>
  CC      drivers/regulator/msm_gfx_ldo.o<br>
  CC      drivers/rtc/rtc-sysfs.o<br>
  CC      drivers/media/rc/keymaps/rc-tivo.o<br>
  CC      drivers/regulator/rpm-smd-regulator.o<br>
  CC      drivers/spi/spidev.o<br>
  CC      drivers/scsi/scsicam.o<br>
  CC      drivers/rtc/qpnp-rtc.o<br>
  CC      drivers/soc/qcom/watchdog_v2.o<br>
  CC      drivers/soc/qcom/common_log.o<br>
  CC      drivers/soc/qcom/cpu_pwr_ctl.o<br>
  CC      drivers/scsi/scsi_error.o<br>
  LD      drivers/rtc/rtc-core.o<br>
  CC      drivers/media/rc/keymaps/rc-total-media-in-hand.o<br>
  CC      drivers/regulator/qpnp-regulator.o<br>
  LD      drivers/soundwire/built-in.o<br>
  CC      drivers/staging/staging.o<br>
  CC      drivers/switch/switch_class.o<br>
  CC      drivers/soc/qcom/socinfo.o<br>
  CC      drivers/spi/spi-qup.o<br>
  CC      drivers/power/qcom/pm-boot.o<br>
  CC      drivers/soc/qcom/boot_stats.o<br>
  CC      drivers/soc/qcom/rpm-smd.o<br>
  CC      drivers/media/rc/keymaps/rc-total-media-in-hand-02.o<br>
  CC      drivers/scsi/scsi_lib.o<br>
  LD      drivers/rtc/built-in.o<br>
  CC      drivers/power/qcom/msm-core.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_qmi_service_v01.o<br>
  CC      drivers/thermal/thermal_core.o<br>
  CC      drivers/staging/android/ion/ion.o<br>
  LD      drivers/switch/built-in.o<br>
  CC      drivers/thermal/thermal_hwmon.o<br>
  CC      drivers/staging/android/binder.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_qmi_service.o<br>
  CC      drivers/spi/spi_qsd.o<br>
  CC      drivers/staging/android/binder_alloc.o<br>
  CC      drivers/media/rc/keymaps/rc-trekstor.o<br>
  CC      drivers/regulator/spm-regulator.o<br>
  CC      drivers/soc/qcom/event_timer.o<br>
  CC      drivers/soc/qcom/rpm-smd-debug.o<br>
  CC      drivers/media/rc/keymaps/rc-tt-1500.o<br>
  CC      drivers/soc/qcom/spm.o<br>
  CC      drivers/scsi/scsi_lib_dma.o<br>
  CC      drivers/staging/android/ashmem.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/rmnet_ipa_fd_ioctl.o<br>
  CC      drivers/media/rc/keymaps/rc-twinhan1027.o<br>
  CC      drivers/regulator/cpr-regulator.o<br>
  CC      drivers/staging/snappy/csnappy_compress.o<br>
  CC      drivers/regulator/cpr3-regulator.o<br>
  CC      drivers/media/rc/keymaps/rc-videomate-m1f.o<br>
  LD      drivers/staging/prima/built-in.o<br>
  CC      drivers/power/qcom/debug_core.o<br>
  CC      drivers/power/qcom/apm.o<br>
  CC      drivers/thermal/of-thermal.o<br>
  CC      drivers/soc/qcom/spm_devices.o<br>
  CC      drivers/scsi/scsi_scan.o<br>
  CC      drivers/staging/android/ion/ion_heap.o<br>
  CC      drivers/soc/qcom/scm.o<br>
  CC      drivers/soc/qcom/scm-boot.o<br>
  CC      drivers/media/rc/keymaps/rc-videomate-s350.o<br>
  LD      drivers/spi/built-in.o<br>
  CC      drivers/staging/snappy/csnappy_decompress.o<br>
  LD      drivers/platform/msm/ipa/ipa_v2/ipat.o<br>
  CC      drivers/tty/tty_io.o<br>
  LD      drivers/platform/msm/ipa/ipa_v2/built-in.o<br>
  CC      drivers/thermal/step_wise.o<br>
  LD      drivers/platform/msm/ipa/built-in.o<br>
  LD      drivers/platform/msm/built-in.o<br>
  LD      drivers/platform/built-in.o<br>
  CC      drivers/tty/n_tty.o<br>
  CC      drivers/media/rc/keymaps/rc-videomate-tv-pvr.o<br>
  LD      drivers/power/qcom/built-in.o<br>
  LD      drivers/power/built-in.o<br>
  CC      drivers/uio/uio.o<br>
  CC      drivers/tty/tty_ioctl.o<br>
  CC      drivers/tty/tty_ldisc.o<br>
  CC      drivers/staging/android/ion/ion_page_pool.o<br>
  CC      drivers/tty/tty_buffer.o<br>
  LD      drivers/staging/snappy/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-winfast.o<br>
  CC      drivers/thermal/msm-tsens.o<br>
  CC      drivers/tty/tty_port.o<br>
  CC      drivers/media/rc/keymaps/rc-winfast-usbii-deluxe.o<br>
  CC      drivers/soc/qcom/mpm-of.o<br>
  CC      drivers/soc/qcom/smem.o<br>
  CC      drivers/staging/android/ion/ion_system_heap.o<br>
  CC      drivers/scsi/scsi_sysfs.o<br>
  CC      drivers/media/rc/keymaps/rc-su3000.o<br>
  CC      drivers/staging/android/ion/ion_carveout_heap.o<br>
  CC      drivers/uio/msm_sharedmem/msm_sharedmem.o<br>
  CC      drivers/uio/msm_sharedmem/remote_filesystem_access_v01.o<br>
  CC      drivers/soc/qcom/smem_debug.o<br>
  CC      drivers/tty/tty_mutex.o<br>
  CC      drivers/staging/android/logger.o<br>
  CC      drivers/thermal/qpnp-temp-alarm.o<br>
  LD      drivers/media/rc/keymaps/built-in.o<br>
  CC      drivers/thermal/qpnp-adc-tm.o<br>
  LD      drivers/media/rc/built-in.o<br>
  CC      drivers/staging/android/ion/ion_chunk_heap.o<br>
  LD      drivers/media/built-in.o<br>
  CC      drivers/uio/msm_sharedmem/sharedmem_qmi.o<br>
  CC      drivers/soc/qcom/smd.o<br>
  CC      drivers/usb/class/cdc-acm.o<br>
  CC      drivers/video/console/dummycon.o<br>
  CC      drivers/tty/tty_ldsem.o<br>
  CC      drivers/usb/common/common.o<br>
  CC      drivers/thermal/msm_thermal.o<br>
  CC      drivers/usb/core/usb.o<br>
  CC      drivers/video/fbdev/core/fb_notify.o<br>
  CC      drivers/usb/core/hub.o<br>
  CC      drivers/video/fbdev/core/fb_cmdline.o<br>
  CC      drivers/staging/android/ion/ion_system_secure_heap.o<br>
  CC      drivers/scsi/scsi_devinfo.o<br>
  LD      drivers/uio/msm_sharedmem/built-in.o<br>
  CC      drivers/tty/pty.o<br>
  LD      drivers/uio/built-in.o<br>
  CC      drivers/tty/tty_audit.o<br>
  LD      drivers/video/console/built-in.o<br>
  CC      drivers/tty/sysrq.o<br>
  CC      drivers/regulator/cpr3-util.o<br>
  LD      drivers/usb/common/usb-common.o<br>
  LD      drivers/usb/common/built-in.o<br>
  CC      drivers/video/fbdev/core/fbmem.o<br>
  CC      drivers/usb/dwc3/core.o<br>
  CC      drivers/thermal/msm_thermal-dev.o<br>
  CC      drivers/video/fbdev/core/fbmon.o<br>
  CC      drivers/staging/android/ion/ion_cma_heap.o<br>
  LD      drivers/usb/class/built-in.o<br>
  CC      drivers/staging/android/timed_output.o<br>
  CC      drivers/scsi/scsi_sysctl.o<br>
  CC      drivers/usb/gadget/usbstring.o<br>
  CC      drivers/usb/gadget/config.o<br>
  CC      drivers/usb/gadget/epautoconf.o<br>
  CC      drivers/usb/core/hcd.o<br>
  CC      drivers/soc/qcom/smd_debug.o<br>
  CC      drivers/regulator/cpr3-hmss-regulator.o<br>
  CC      drivers/staging/android/ion/ion_cma_secure_heap.o<br>
  CC      drivers/tty/serial/serial_core.o<br>
  CC      drivers/scsi/scsi_proc.o<br>
  CC      drivers/regulator/cpr3-mmss-regulator.o<br>
  CC      drivers/tty/vt/vt_ioctl.o<br>
  CC      drivers/scsi/scsi_trace.o<br>
  CC      drivers/staging/android/timed_gpio.o<br>
  CC      drivers/usb/gadget/composite.o<br>
  CC      drivers/usb/gadget/functions.o<br>
  CC      drivers/video/fbdev/core/fbcmap.o<br>
  CC      drivers/usb/dwc3/debug.o<br>
  CC      drivers/soc/qcom/smd_private.o<br>
  CC      drivers/soc/qcom/smd_init_dt.o<br>
  CC      drivers/scsi/scsi_pm.o<br>
  CC      drivers/usb/core/urb.o<br>
  CC      drivers/usb/dwc3/trace.o<br>
  CC      drivers/staging/android/ion/compat_ion.o<br>
  CC      drivers/soc/qcom/smsm_debug.o<br>
  CC      drivers/usb/gadget/u_f.o<br>
  CC      drivers/video/fbdev/msm/../../msm/msm_dba/msm_dba.o<br>
  CC      drivers/regulator/cpr4-apss-regulator.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp.o<br>
  CC      drivers/video/fbdev/core/fbsysfs.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_ctl.o<br>
  CC      drivers/tty/vt/vc_screen.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pipe.o<br>
  CC      drivers/staging/android/ion/msm/msm_ion.o<br>
  CC      drivers/staging/android/ion/msm/compat_msm_ion.o<br>
  CC      drivers/soc/qcom/glink.o<br>
  CC      drivers/tty/serial/msm_serial_hs.o<br>
  CC      drivers/video/fbdev/msm/../../msm/msm_dba/msm_dba_init.o<br>
  CC      drivers/scsi/ufs/ufs-qcom.o<br>
  CC      drivers/usb/gadget/debug.o<br>
  CC      drivers/usb/dwc3/host.o<br>
  CC      drivers/usb/core/message.o<br>
  CC      drivers/video/fbdev/msm/../../msm/msm_dba/msm_dba_helpers.o<br>
  CC      drivers/video/fbdev/core/modedb.o<br>
  CC      drivers/tty/vt/selection.o<br>
  CC      drivers/regulator/cprh-kbss-regulator.o<br>
  CC      drivers/scsi/ufs/ufs-qcom-ice.o<br>
  CC      drivers/thermal/lmh_interface.o<br>
  CC      drivers/usb/dwc3/gadget.o<br>
  CC      drivers/usb/gadget/function/f_acm.o<br>
  LD      drivers/staging/android/ion/msm/built-in.o<br>
  LD      drivers/staging/android/ion/built-in.o<br>
  CC      drivers/video/fbdev/msm/../../msm/msm_dba/msm_dba_debug.o<br>
  CC      drivers/staging/android/sync.o<br>
  CC      drivers/tty/vt/keyboard.o<br>
  CC      drivers/scsi/ufs/ufshcd.o<br>
  CC      drivers/video/fbdev/core/fbcvt.o<br>
  CC      drivers/regulator/qpnp-labibb-regulator.o<br>
  CC      drivers/video/fbdev/msm/../../msm/msm_dba/adv7533.o<br>
  CC      drivers/scsi/ufs/ufs_quirks.o<br>
  CC      drivers/usb/core/driver.o<br>
  CC      drivers/thermal/lmh_lite.o<br>
  CC      drivers/usb/gadget/function/u_serial.o<br>
  CC      drivers/usb/gadget/function/f_serial.o<br>
  CC      drivers/video/fbdev/core/cfbfillrect.o<br>
  CC      drivers/video/fbdev/core/cfbcopyarea.o<br>
  CC      drivers/tty/serial/msm_smd_tty.o<br>
  CC      drivers/staging/android/sw_sync.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_util.o<br>
  LD      drivers/video/fbdev/msm/../../msm/msm_dba/built-in.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/dsi_status_6g.o<br>
  CC      drivers/tty/vt/consolemap.o<br>
  CONMK   drivers/tty/vt/consolemap_deftbl.c<br>
  CC      drivers/scsi/sd.o<br>
  CC      drivers/regulator/stub-regulator.o<br>
  CC      drivers/usb/dwc3/ep0.o<br>
  CC      drivers/usb/core/config.o<br>
  CC      drivers/usb/gadget/function/f_ncm.o<br>
  CC      drivers/regulator/kryo-regulator.o<br>
  CC      drivers/video/fbdev/core/cfbimgblt.o<br>
  CC      drivers/staging/android/oneshot_sync.o<br>
  CC      drivers/usb/host/pci-quirks.o<br>
  LD      drivers/thermal/thermal_sys.o<br>
  LD      drivers/thermal/built-in.o<br>
  CC      drivers/soc/qcom/glink_debugfs.o<br>
  CC      drivers/scsi/sg.o<br>
  LD      drivers/tty/serial/built-in.o<br>
  CC      drivers/usb/host/xhci-pci.o<br>
  CC      drivers/soc/qcom/glink_ssr.o<br>
  CC      drivers/usb/gadget/function/f_ecm.o<br>
  CC      drivers/tty/vt/vt.o<br>
  CC      drivers/usb/dwc3/debugfs.o<br>
  CC      drivers/usb/core/file.o<br>
  LD      drivers/video/fbdev/core/fb.o<br>
  LD      drivers/video/fbdev/core/built-in.o<br>
  LD      drivers/regulator/built-in.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp.o<br>
  CC      drivers/usb/gadget/function/f_mass_storage.o<br>
  LD      drivers/staging/android/built-in.o<br>
  LD      drivers/staging/built-in.o<br>
  CC      drivers/usb/core/buffer.o<br>
  CC      drivers/usb/gadget/function/storage_common.o<br>
  CC      drivers/usb/dwc3/dwc3-pci.o<br>
  CC      drivers/usb/dwc3/dwc3-msm.o<br>
  CC      drivers/soc/qcom/glink_loopback_server.o<br>
  CC      drivers/usb/host/xhci-plat.o<br>
  CC      drivers/usb/host/ehci-hcd.o<br>
  CC      drivers/usb/core/sysfs.o<br>
  CC      drivers/usb/host/ehci-pci.o<br>
  CC      drivers/usb/gadget/function/f_fs.o<br>
  CC      drivers/usb/dwc3/dbm.o<br>
  CC      drivers/usb/gadget/function/f_uac1.o<br>
  CC      drivers/scsi/ch.o<br>
  CC      drivers/soc/qcom/glink_smd_xprt.o<br>
  CC      drivers/usb/host/ehci-msm.o<br>
  CC      drivers/usb/host/xhci.o<br>
  CC      drivers/soc/qcom/glink_smem_native_xprt.o<br>
  CC      drivers/usb/host/xhci-mem.o<br>
  LD      drivers/usb/dwc3/dwc3.o<br>
  CC      drivers/soc/qcom/smem_log.o<br>
  CC      drivers/usb/core/endpoint.o<br>
  CC      drivers/usb/host/xhci-ring.o<br>
  CC      drivers/usb/gadget/function/u_uac1.o<br>
  CC      drivers/usb/gadget/function/f_uac2.o<br>
  CC      drivers/usb/gadget/function/f_uvc.o<br>
  CC      drivers/scsi/ufs/ufshcd-pltfrm.o<br>
  CC      drivers/usb/core/devio.o<br>
  CC      drivers/usb/core/notify.o<br>
  LD      drivers/usb/dwc3/built-in.o<br>
  CC      drivers/usb/gadget/function/uvc_queue.o<br>
  CC      drivers/soc/qcom/smp2p.o<br>
  SHIPPED drivers/tty/vt/defkeymap.c<br>
  CC      drivers/tty/vt/consolemap_deftbl.o<br>
  CC      drivers/tty/vt/defkeymap.o<br>
  LD      drivers/tty/vt/built-in.o<br>
  LD      drivers/tty/built-in.o<br>
  CC      drivers/soc/qcom/smp2p_debug.o<br>
  CC      drivers/soc/qcom/smp2p_sleepstate.o<br>
  CC      drivers/usb/gadget/function/uvc_v4l2.o<br>
  CC      drivers/usb/gadget/function/uvc_video.o<br>
  CC      drivers/scsi/ufs/ufs-debugfs.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_debug.o<br>
  CC      drivers/usb/gadget/function/f_audio_source.o<br>
  CC      drivers/usb/gadget/function/f_hid.o<br>
  CC      drivers/usb/host/xhci-hub.o<br>
  CC      drivers/usb/host/xhci-dbg.o<br>
  CC      drivers/usb/misc/ehset.o<br>
  CC      drivers/usb/mon/mon_main.o<br>
  CC      drivers/usb/host/xhci-trace.o<br>
  CC      drivers/usb/core/generic.o<br>
  CC      drivers/usb/core/quirks.o<br>
  CC      drivers/usb/core/devices.o<br>
  CC      drivers/soc/qcom/smp2p_loopback.o<br>
  CC      drivers/soc/qcom/smp2p_test.o<br>
  LD      drivers/usb/misc/built-in.o<br>
  LD      drivers/usb/gadget/function/usb_f_acm.o<br>
  LD      drivers/usb/gadget/function/usb_f_serial.o<br>
  LD      drivers/usb/gadget/function/usb_f_ncm.o<br>
  LD      drivers/usb/gadget/function/usb_f_ecm.o<br>
  LD      drivers/usb/gadget/function/usb_f_mass_storage.o<br>
  CC      drivers/usb/serial/usb-serial.o<br>
  LD      drivers/usb/gadget/function/usb_f_fs.o<br>
  LD      drivers/usb/host/xhci-plat-hcd.o<br>
  LD      drivers/usb/gadget/function/usb_f_uac1.o<br>
  CC      drivers/scsi/ufs/ufs-qcom-debugfs.o<br>
  LD      drivers/usb/gadget/function/usb_f_uac2.o<br>
  LD      drivers/usb/gadget/function/usb_f_uvc.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_cache_config.o<br>
  CC      drivers/usb/phy/phy.o<br>
  LD      drivers/usb/gadget/function/usb_f_audio_source.o<br>
  CC      drivers/usb/core/port.o<br>
  LD      drivers/usb/gadget/function/usb_f_hid.o<br>
  LD      drivers/usb/gadget/function/built-in.o<br>
  CC      drivers/usb/mon/mon_stat.o<br>
  CC      drivers/usb/gadget/udc/udc-core.o<br>
  CC      drivers/usb/serial/generic.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_intf_video.o<br>
  LD      drivers/scsi/scsi_mod.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_intf_cmd.o<br>
  CC      drivers/usb/gadget/android.o<br>
  LD      drivers/usb/host/xhci-hcd.o<br>
  CC      drivers/usb/core/hcd-pci.o<br>
  CC      drivers/usb/gadget/ci13xxx_msm.o<br>
  CC      drivers/soc/qcom/smp2p_spinlock_test.o<br>
  CC      drivers/usb/mon/mon_text.o<br>
  LD      drivers/scsi/ufs/built-in.o<br>
  CC      drivers/usb/mon/mon_bin.o<br>
  LD      drivers/scsi/sd_mod.o<br>
  LD      drivers/scsi/built-in.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_intf_writeback.o<br>
  CC      drivers/usb/phy/of.o<br>
  CC      drivers/usb/phy/class-dual-role.o<br>
  LD      drivers/usb/host/built-in.o<br>
  LD      drivers/usb/gadget/udc/built-in.o<br>
  LD      drivers/usb/gadget/libcomposite.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_rotator.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_overlay.o<br>
  CC      drivers/usb/serial/bus.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_layer.o<br>
  CC      drivers/soc/qcom/qmi_interface.o<br>
  CC      drivers/usb/storage/scsiglue.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_splash_logo.o<br>
  LD      drivers/usb/core/usbcore.o<br>
  LD      drivers/usb/core/built-in.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_cdm.o<br>
  CC      drivers/usb/phy/phy-generic.o<br>
  CC      drivers/soc/qcom/ipc_router_smd_xprt.o<br>
  CC      drivers/usb/phy/phy-msm-usb.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_smmu.o<br>
  LD      drivers/usb/mon/usbmon.o<br>
  LD      drivers/usb/mon/built-in.o<br>
  LD      drivers/usb/serial/usbserial.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_wfd.o<br>
  LD      drivers/usb/serial/built-in.o<br>
  CC      drivers/soc/qcom/memshare/heap_mem_ext_v01.o<br>
  CC      drivers/soc/qcom/memshare/msm_memshare.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_v1_7.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_v3.o<br>
  CC      drivers/usb/storage/protocol.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_common.o<br>
  CC      drivers/usb/phy/phy-msm-hsusb.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_debug.o<br>
  CC      drivers/soc/qcom/rpm_rbcpr_stats_v2.o<br>
  CC      drivers/soc/qcom/qdsp6v2/apr.o<br>
  CC      drivers/soc/qcom/qdsp6v2/apr_v2.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_debug.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_debug_xlog.o<br>
  LD      drivers/soc/qcom/memshare/built-in.o<br>
  CC      drivers/soc/qcom/cpaccess64.o<br>
  CC      drivers/soc/qcom/qdsp6v2/apr_tal.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi.o<br>
  CC      drivers/usb/storage/transport.o<br>
  CC      drivers/usb/storage/usb.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_host.o<br>
  CC      drivers/soc/qcom/rpm_stats.o<br>
  CC      drivers/usb/phy/phy-msm-ssusb-qmp.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_cmd.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_status.o<br>
  CC      drivers/usb/storage/initializers.o<br>
  CC      drivers/soc/qcom/qdsp6v2/voice_svc.o<br>
  CC      drivers/soc/qcom/qdsp6v2/msm_audio_ion.o<br>
  CC      drivers/soc/qcom/qdsp6v2/adsp-loader.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_panel.o<br>
  CC      drivers/usb/phy/phy-msm-qusb.o<br>
  CC      drivers/soc/qcom/rpm_master_stat.o<br>
  CC      drivers/usb/storage/sierra_ms.o<br>
  CC      drivers/usb/storage/option_ms.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/msm_mdss_io_8974.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_phy.o<br>
  CC      drivers/usb/storage/usual-tables.o<br>
  CC      drivers/soc/qcom/rpm_rail_stats.o<br>
  CC      drivers/soc/qcom/system_stats.o<br>
  CC      drivers/soc/qcom/perf_event_l2.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_clk.o<br>
  CC      drivers/soc/qcom/rpm_log.o<br>
  CC      drivers/usb/storage/alauda.o<br>
  CC      drivers/usb/storage/cypress_atacb.o<br>
  CC      drivers/soc/qcom/msm_tz_smmu.o<br>
  CC      drivers/soc/qcom/peripheral-loader.o<br>
  LD      drivers/soc/qcom/qdsp6v2/built-in.o<br>
  CC      drivers/soc/qcom/subsys-pil-tz.o<br>
  CC      drivers/usb/storage/datafab.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_panel.o<br>
  LD      drivers/usb/phy/built-in.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_util.o<br>
  CC      drivers/usb/storage/freecom.o<br>
  CC      drivers/usb/storage/isd200.o<br>
  CC      drivers/usb/storage/jumpshot.o<br>
  CC      drivers/usb/storage/karma.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_edid.o<br>
  CC      drivers/soc/qcom/pil-q6v5.o<br>
  CC      drivers/soc/qcom/pil-msa.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_cec_core.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dba_utils.o<br>
  CC      drivers/usb/storage/sddr09.o<br>
  CC      drivers/usb/storage/sddr55.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_io_util.o<br>
  CC      drivers/soc/qcom/pil-q6v5-mss.o<br>
  CC      drivers/usb/storage/shuttle_usbat.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_tx.o<br>
  LD      drivers/usb/storage/usb-storage.o<br>
  LD      drivers/usb/storage/ums-alauda.o<br>
  LD      drivers/usb/storage/ums-cypress.o<br>
  LD      drivers/usb/storage/ums-datafab.o<br>
  LD      drivers/usb/storage/ums-freecom.o<br>
  LD      drivers/usb/storage/ums-isd200.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_panel.o<br>
  LD      drivers/usb/storage/ums-jumpshot.o<br>
  LD      drivers/usb/storage/ums-karma.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_hdcp.o<br>
  CC      drivers/soc/qcom/msm_performance.o<br>
  CC      drivers/soc/qcom/subsystem_notif.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_hdcp2p2.o<br>
  CC      drivers/soc/qcom/subsystem_restart.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_cec.o<br>
  CC      drivers/soc/qcom/ramdump.o<br>
  CC      drivers/soc/qcom/sysmon.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_audio.o<br>
  LD      drivers/usb/storage/ums-sddr55.o<br>
  CC      drivers/soc/qcom/sysmon-qmi.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_wb.o<br>
  LD      drivers/usb/storage/ums-sddr09.o<br>
  CC      drivers/soc/qcom/secure_buffer.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_fb.o<br>
  LD      drivers/usb/storage/ums-usbat.o<br>
  LD      drivers/usb/storage/built-in.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_util.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_compat_utils.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_kcal_ctrl.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/lcd_notify.o<br>
  CC      drivers/soc/qcom/icnss.o<br>
  LD      drivers/video/fbdev/msm/../../msm/mdss/mdss-mdp.o<br>
  LD      drivers/video/fbdev/msm/../../msm/mdss/mdss-dsi.o<br>
  CC      drivers/soc/qcom/wlan_firmware_service_v01.o<br>
  CC      drivers/soc/qcom/bam_dmux.o<br>
  CC      drivers/soc/qcom/scm-xpu.o<br>
  CC      drivers/soc/qcom/serial_num.o<br>
  CC      drivers/soc/qcom/state_notifier.o<br>
../../../../../../kernel/xiaomi/msm8953/drivers/soc/qcom/msm_performance.c: In function 'set_cpu_min_freq':<br>
../../../../../../kernel/xiaomi/msm8953/drivers/soc/qcom/msm_performance.c:410:2: warning: ISO C90 forbids mixed declarations and code [-Wdeclaration-after-statement]<br>
  const char *reset = "0:0 2:0";<br>
  ^<br>
../../../../../../kernel/xiaomi/msm8953/drivers/soc/qcom/msm_performance.c: At top level:<br>
cc1: warning: unrecognized command line option "-Wno-misleading-indentation"<br>
  LD      drivers/video/fbdev/msm/../../msm/mdss/built-in.o<br>
  LD      drivers/soc/qcom/built-in.o<br>
  LD      drivers/soc/built-in.o<br>
  LD      drivers/video/fbdev/msm/../../msm/built-in.o<br>
  LD      drivers/video/fbdev/msm/built-in.o<br>
  LD      drivers/video/fbdev/built-in.o<br>
  LD      drivers/video/built-in.o<br>
  LD      drivers/usb/gadget/g_android.o<br>
  LD      drivers/usb/gadget/built-in.o<br>
  LD      drivers/usb/built-in.o<br>
  LD      drivers/built-in.o<br>
  LINK    vmlinux<br>
  LD      vmlinux.o<br>
  MODPOST vmlinux.o<br>
  GEN     .version<br>
  CHK     include/generated/compile.h<br>
  UPD     include/generated/compile.h<br>
  CC      init/version.o<br>
  LD      init/built-in.o<br>
  KSYM    .tmp_kallsyms1.o<br>
  KSYM    .tmp_kallsyms2.o<br>
  LD      vmlinux<br>
  SORTEX  vmlinux<br>
  SYSMAP  System.map<br>
  OBJCOPY arch/arm64/boot/Image<br>
  DTC     arch/arm64/boot/dts/qcom/msm8953-qrd-sku3.dtb<br>
  DTC     arch/arm64/boot/dts/qcom/msm8952-qrd-skum.dtb<br>
  DTC     arch/arm64/boot/dts/qcom/msm8952-cdp.dtb<br>
  DTC     arch/arm64/boot/dts/qcom/msm8952-ext-codec-cdp.dtb<br>
  DTC     arch/arm64/boot/dts/qcom/msm8952-mtp.dtb<br>
  GZIP    arch/arm64/boot/Image.gz<br>
  CAT     arch/arm64/boot/Image.gz-dtb<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
Building DTBs<br>
make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  CHK     include/config/kernel.release<br>
  GEN     ./Makefile<br>
  CHK     include/generated/uapi/linux/version.h<br>
  CHK     include/generated/utsrelease.h<br>
  Using /home/stalker/hadk/kernel/xiaomi/msm8953 as source for kernel<br>
  CALL    /home/stalker/hadk/kernel/xiaomi/msm8953/scripts/checksyscalls.sh<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
Building Kernel Modules<br>
make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  CHK     include/config/kernel.release<br>
  GEN     ./Makefile<br>
  CHK     include/generated/uapi/linux/version.h<br>
  CHK     include/generated/utsrelease.h<br>
  Using /home/stalker/hadk/kernel/xiaomi/msm8953 as source for kernel<br>
  CALL    /home/stalker/hadk/kernel/xiaomi/msm8953/scripts/checksyscalls.sh<br>
  CC [M]  fs/autofs4/init.o<br>
  CC [M]  fs/autofs4/inode.o<br>
  CC [M]  drivers/bluetooth/hci_ldisc.o<br>
  CC [M]  fs/autofs4/root.o<br>
  CC [M]  fs/autofs4/symlink.o<br>
  CC [M]  drivers/bluetooth/hci_h4.o<br>
  CC [M]  drivers/bluetooth/bluetooth-power.o<br>
  CC [M]  fs/autofs4/waitq.o<br>
  CC [M]  net/bluetooth/af_bluetooth.o<br>
  CC [M]  fs/autofs4/expire.o<br>
  CC [M]  net/bluetooth/hci_core.o<br>
  CC [M]  net/bluetooth/hci_conn.o<br>
  CC [M]  net/bluetooth/hci_event.o<br>
  CC [M]  fs/autofs4/dev-ioctl.o<br>
  CC [M]  net/bluetooth/mgmt.o<br>
  CC [M]  net/bluetooth/hci_sock.o<br>
  CC [M]  net/bluetooth/hci_sysfs.o<br>
  CC [M]  net/bluetooth/l2cap_core.o<br>
  CC [M]  fs/overlayfs/super.o<br>
  CC [M]  net/bluetooth/l2cap_sock.o<br>
  LD [M]  drivers/bluetooth/hci_uart.o<br>
  CC [M]  fs/overlayfs/inode.o<br>
  CC [M]  net/ipv4/tcp_bic.o<br>
  CC [M]  net/ipv4/tcp_westwood.o<br>
  LD [M]  fs/autofs4/autofs4.o<br>
  CC [M]  fs/overlayfs/dir.o<br>
  CC [M]  fs/overlayfs/readdir.o<br>
  CC [M]  fs/overlayfs/copy_up.o<br>
  CC [M]  net/bluetooth/smp.o<br>
  CC [M]  net/bluetooth/sco.o<br>
  CC [M]  net/bluetooth/lib.o<br>
  CC [M]  net/bluetooth/a2mp.o<br>
  LD [M]  fs/overlayfs/overlay.o<br>
  CC [M]  net/bluetooth/amp.o<br>
  CC [M]  net/netfilter/nfnetlink_acct.o<br>
  CC [M]  net/netfilter/xt_nfacct.o<br>
  CC [M]  net/bluetooth/bnep/core.o<br>
  CC [M]  net/bluetooth/hidp/core.o<br>
  CC [M]  net/bluetooth/hidp/sock.o<br>
  CC [M]  net/bluetooth/bnep/sock.o<br>
  CC [M]  net/bluetooth/rfcomm/core.o<br>
  CC [M]  net/bluetooth/rfcomm/sock.o<br>
  CC [M]  net/bluetooth/rfcomm/tty.o<br>
  CC [M]  net/bluetooth/bnep/netdev.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/main.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/netdev.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/cfg80211.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/pcie_bus.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/debugfs.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/wmi.o<br>
  LD [M]  net/bluetooth/hidp/hidp.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/interrupt.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/txrx.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/debug.o<br>
  LD [M]  net/bluetooth/bluetooth.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/rx_reorder.o<br>
  LD [M]  net/bluetooth/bnep/bnep.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/ioctl.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/fw.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/pm.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/pmc.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/trace.o<br>
  LD [M]  net/bluetooth/rfcomm/rfcomm.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/wil_platform.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/ethtool.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/wil_crash_dump.o<br>
  CC [M]  drivers/net/wireless/ath/wil6210/p2p.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapApiData.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapApiDebug.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapApiExt.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapApiHCBB.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapApiInfo.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapApiLinkCntl.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapApiLinkSupervision.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapApiStatus.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapApiTimer.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapModule.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapRsn8021xAuthFsm.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapRsn8021xPrf.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapRsn8021xSuppRsnFsm.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapRsnAsfPacket.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapRsnSsmAesKeyWrap.o<br>
  LD [M]  drivers/net/wireless/ath/wil6210/wil6210.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapRsnSsmEapol.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapRsnSsmReplayCtr.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/bapRsnTxRx.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/btampFsm.o<br>
  CC [M]  drivers/staging/prima/CORE/BAP/src/btampHCI.o<br>
  CC [M]  drivers/staging/prima/CORE/DXE/src/wlan_qct_dxe.o<br>
  CC [M]  drivers/staging/prima/CORE/DXE/src/wlan_qct_dxe_cfg_i.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/bap_hdd_main.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_assoc.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_cfg.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_debugfs.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_dev_pwr.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_dp_utils.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_early_suspend.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_ftm.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_hostapd.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_main.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_oemdata.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_mib.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_scan.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_softap_tx_rx.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_tx_rx.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_trace.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_wext.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_wmm.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_wowl.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_cfg80211.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_p2p.o<br>
  CC [M]  drivers/staging/prima/CORE/HDD/src/wlan_hdd_tdls.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/cfg/cfgApi.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/cfg/cfgDebug.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/cfg/cfgParamName.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/cfg/cfgProcMsg.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/cfg/cfgSendMsg.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/dph/dphHashTable.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limAIDmgmt.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limAdmitControl.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limApi.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limAssocUtils.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limDebug.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limFT.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limIbssPeerMgmt.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limLinkMonitoringAlgo.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limLogDump.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limP2P.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessActionFrame.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessAssocReqFrame.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessAssocRspFrame.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessAuthFrame.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessBeaconFrame.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessCfgUpdates.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessDeauthFrame.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessDisassocFrame.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessLmmMessages.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessMessageQueue.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessMlmReqMessages.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessMlmRspMessages.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessProbeReqFrame.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessProbeRspFrame.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessSmeReqMessages.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limPropExtsUtils.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limRMC.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limRoamingAlgo.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limScanResultUtils.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limSecurityUtils.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limSendManagementFrames.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limSendMessages.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limSendSmeRspMessages.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limSerDesUtils.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limSession.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limSessionUtils.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limSmeReqUtils.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limStaHashApi.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limTimerUtils.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limTrace.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limUtils.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessTdls.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/pmm/pmmAP.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/pmm/pmmApi.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/pmm/pmmDebug.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/sch/schApi.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/sch/schBeaconGen.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/sch/schBeaconProcess.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/sch/schDebug.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/sch/schMessage.o<br>
  CC [M]  drivers/staging/prima/CORE/MAC/src/pe/rrm/rrmApi.o<br>
  CC [M]  drivers/staging/prima/CORE/SAP/src/sapApiLinkCntl.o<br>
  CC [M]  drivers/staging/prima/CORE/SAP/src/sapChSelect.o<br>
  CC [M]  drivers/staging/prima/CORE/SAP/src/sapFsm.o<br>
  CC [M]  drivers/staging/prima/CORE/SAP/src/sapModule.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/btc/btcApi.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/ccm/ccmApi.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/ccm/ccmLogDump.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/sme_common/sme_Api.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/sme_common/sme_FTApi.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/sme_common/sme_Trace.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/csr/csrApiRoam.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/csr/csrApiScan.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/csr/csrCmdProcess.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/csr/csrLinkList.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/csr/csrLogDump.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/csr/csrNeighborRoam.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/csr/csrUtil.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/csr/csrTdlsProcess.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/oemData/oemDataApi.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/p2p/p2p_Api.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/pmc/pmcApi.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/pmc/pmc.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/pmc/pmcLogDump.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/QoS/sme_Qos.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/rrm/sme_rrm.o<br>
  CC [M]  drivers/staging/prima/CORE/SME/src/nan/nan_Api.o<br>
  CC [M]  drivers/staging/prima/CORE/SVC/src/btc/wlan_btc_svc.o<br>
  CC [M]  drivers/staging/prima/CORE/SVC/src/nlink/wlan_nlink_srv.o<br>
  CC [M]  drivers/staging/prima/CORE/SVC/src/ptt/wlan_ptt_sock_svc.o<br>
  CC [M]  drivers/staging/prima/CORE/SVC/src/logging/wlan_logging_sock_svc.o<br>
  CC [M]  drivers/staging/prima/CORE/SYS/common/src/wlan_qct_sys.o<br>
  CC [M]  drivers/staging/prima/CORE/SYS/legacy/src/pal/src/palApiComm.o<br>
  CC [M]  drivers/staging/prima/CORE/SYS/legacy/src/pal/src/palTimer.o<br>
  CC [M]  drivers/staging/prima/CORE/SYS/legacy/src/platform/src/VossWrapper.o<br>
  CC [M]  drivers/staging/prima/CORE/SYS/legacy/src/system/src/macInitApi.o<br>
  CC [M]  drivers/staging/prima/CORE/SYS/legacy/src/system/src/sysEntryFunc.o<br>
  CC [M]  drivers/staging/prima/CORE/SYS/legacy/src/utils/src/dot11f.o<br>
  CC [M]  drivers/staging/prima/CORE/SYS/legacy/src/utils/src/logApi.o<br>
  CC [M]  drivers/staging/prima/CORE/SYS/legacy/src/utils/src/logDump.o<br>
  CC [M]  drivers/staging/prima/CORE/SYS/legacy/src/utils/src/macTrace.o<br>
  CC [M]  drivers/staging/prima/CORE/SYS/legacy/src/utils/src/parserApi.o<br>
  CC [M]  drivers/staging/prima/CORE/SYS/legacy/src/utils/src/utilsApi.o<br>
  CC [M]  drivers/staging/prima/CORE/SYS/legacy/src/utils/src/utilsParser.o<br>
  CC [M]  drivers/staging/prima/CORE/TL/src/wlan_qct_tl.o<br>
  CC [M]  drivers/staging/prima/CORE/TL/src/wlan_qct_tl_ba.o<br>
  CC [M]  drivers/staging/prima/CORE/TL/src/wlan_qct_tl_hosupport.o<br>
  CC [M]  drivers/staging/prima/CORE/TL/src/wlan_qct_tl_trace.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_api.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_event.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_getBin.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_list.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_lock.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_memory.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_mq.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_nvitem.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_packet.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_sched.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_threads.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_timer.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_trace.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_types.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_utils.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/wlan_nv_parser.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/wlan_nv_stream_read.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/wlan_nv_template_builtin.o<br>
  CC [M]  drivers/staging/prima/CORE/VOSS/src/vos_diag.o<br>
  CC [M]  drivers/staging/prima/CORE/WDA/src/wlan_qct_wda.o<br>
  CC [M]  drivers/staging/prima/CORE/WDA/src/wlan_qct_wda_debug.o<br>
  CC [M]  drivers/staging/prima/CORE/WDA/src/wlan_qct_wda_ds.o<br>
  CC [M]  drivers/staging/prima/CORE/WDA/src/wlan_qct_wda_legacy.o<br>
  CC [M]  drivers/staging/prima/CORE/WDA/src/wlan_nv.o<br>
  CC [M]  drivers/staging/prima/CORE/WDI/CP/src/wlan_qct_wdi.o<br>
  CC [M]  drivers/staging/prima/CORE/WDI/CP/src/wlan_qct_wdi_dp.o<br>
  CC [M]  drivers/staging/prima/CORE/WDI/CP/src/wlan_qct_wdi_sta.o<br>
  CC [M]  drivers/staging/prima/CORE/WDI/DP/src/wlan_qct_wdi_bd.o<br>
  CC [M]  drivers/staging/prima/CORE/WDI/DP/src/wlan_qct_wdi_ds.o<br>
  CC [M]  drivers/staging/prima/CORE/WDI/TRP/CTS/src/wlan_qct_wdi_cts.o<br>
  CC [M]  drivers/staging/prima/CORE/WDI/TRP/DTS/src/wlan_qct_wdi_dts.o<br>
  CC [M]  drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_api.o<br>
  CC [M]  drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_device.o<br>
  CC [M]  drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_msg.o<br>
  CC [M]  drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_packet.o<br>
  CC [M]  drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_sync.o<br>
  CC [M]  drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_timer.o<br>
  CC [M]  drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_trace.o<br>
  LD [M]  drivers/staging/prima/wlan.o<br>
  Building modules, stage 2.<br>
  MODPOST 14 modules<br>
  CC      drivers/bluetooth/bluetooth-power.mod.o<br>
  CC      drivers/bluetooth/hci_uart.mod.o<br>
  CC      drivers/net/wireless/ath/wil6210/wil6210.mod.o<br>
  CC      drivers/staging/prima/wlan.mod.o<br>
  CC      fs/autofs4/autofs4.mod.o<br>
  CC      fs/overlayfs/overlay.mod.o<br>
  CC      net/bluetooth/bluetooth.mod.o<br>
  CC      net/bluetooth/bnep/bnep.mod.o<br>
  CC      net/bluetooth/hidp/hidp.mod.o<br>
  CC      net/bluetooth/rfcomm/rfcomm.mod.o<br>
  CC      net/ipv4/tcp_bic.mod.o<br>
  CC      net/ipv4/tcp_westwood.mod.o<br>
  CC      net/netfilter/nfnetlink_acct.mod.o<br>
  CC      net/netfilter/xt_nfacct.mod.o<br>
  LD [M]  drivers/staging/prima/wlan.ko<br>
  LD [M]  net/ipv4/tcp_westwood.ko<br>
  LD [M]  drivers/bluetooth/bluetooth-power.ko<br>
  LD [M]  net/netfilter/nfnetlink_acct.ko<br>
  LD [M]  drivers/bluetooth/hci_uart.ko<br>
  LD [M]  net/bluetooth/bluetooth.ko<br>
  LD [M]  net/ipv4/tcp_bic.ko<br>
  LD [M]  drivers/net/wireless/ath/wil6210/wil6210.ko<br>
  LD [M]  net/netfilter/xt_nfacct.ko<br>
  LD [M]  fs/autofs4/autofs4.ko<br>
  LD [M]  fs/overlayfs/overlay.ko<br>
  LD [M]  net/bluetooth/rfcomm/rfcomm.ko<br>
  LD [M]  net/bluetooth/bnep/bnep.ko<br>
  LD [M]  net/bluetooth/hidp/hidp.ko<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  INSTALL drivers/bluetooth/bluetooth-power.ko<br>
  INSTALL drivers/bluetooth/hci_uart.ko<br>
  INSTALL drivers/net/wireless/ath/wil6210/wil6210.ko<br>
  INSTALL drivers/staging/prima/wlan.ko<br>
  INSTALL fs/autofs4/autofs4.ko<br>
  INSTALL fs/overlayfs/overlay.ko<br>
  INSTALL net/bluetooth/bluetooth.ko<br>
  INSTALL net/bluetooth/bnep/bnep.ko<br>
  INSTALL net/bluetooth/hidp/hidp.ko<br>
  INSTALL net/bluetooth/rfcomm/rfcomm.ko<br>
  INSTALL net/ipv4/tcp_bic.ko<br>
  INSTALL net/ipv4/tcp_westwood.ko<br>
  INSTALL net/netfilter/nfnetlink_acct.ko<br>
  INSTALL net/netfilter/xt_nfacct.ko<br>
  DEPMOD  3.18.105-ElectraBlue-11.0-mido<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
[100% 17/17] Target boot image: /home/stalker/hadk/out/target/product/mido/boot.img<br>
/home/stalker/hadk/out/target/product/mido/boot.img maxsize=68395008 blocksize=135168 total=12767232 reserve=811008<br>
[100% 17/17] Install: /home/stalker/hadk/out/target/product/mido/hybris-recovery.img<br>
<br>
#### make completed successfully (03:25 (mm:ss)) ####<br>
</details><br>


Проверим, загрузится ли система с нашим собранным ядром, перезагрузимся в fastboot, проверим что устройство определилось
```console
HOST:~$ sudo fastboot devices
```
<details>
[sudo] пароль для stalker:<br>
6fcdaa929904	fastboot<br>
</details><br>

Прошьем boot
```console
HOST:~$ sudo fastboot flash boot hadk/out/target/product/mido/hybris-boot.img
```
<details>
target reported max download size of 536870912 bytes<br>
sending 'boot' (11658 KB)...<br>
OKAY [  0.351s]<br>
writing 'boot'...<br>
OKAY [  0.173s]<br>
finished. total time: 0.525s<br>
</details><br>

Не вышло, графика не запускается, разберемся почему наше ядро не работает позже...<br>
