跟一下msm8953音频的dts。

msm8953-audio-mtp.dtsi

```
&int_codec {
	status = "okay";
	qcom,model = "msm8953-snd-card-mtp";
	qcom,msm-hs-micbias-type = "internal";	// 具有内部MICBIAS
	qcom,msm-spk-ext-pa = <&tlmm 96 0>;
	action,enable-aw8738;
	qcom,audio-routing =
		"RX_BIAS", "MCLK",
		"SPK_RX_BIAS", "MCLK",					
		"INT_LDO_H", "MCLK",
		"MIC BIAS Internal1", "Handset Mic",	// 听筒采用内部MICBIAS
		"MIC BIAS Internal2", "Headset Mic",	// 耳机采用内部MICBIAS
		"MIC BIAS Internal3", "Secondary Mic",
		"AMIC1", "MIC BIAS Internal1",
		"AMIC2", "MIC BIAS Internal2",
		"AMIC3", "MIC BIAS Internal3";
};
```

msm-audio.dtsi

```
	int_codec: sound {
		status = "disabled";
		compatible = "qcom,msm8952-audio-codec";
		qcom,model = "msm8952-snd-card-mtp";
		reg = <0xc051000 0x4>,
		      <0xc051004 0x4>,
		      <0xc055000 0x4>,
		      <0xc052000 0x4>;
		reg-names = "csr_gp_io_mux_mic_ctl",
			    "csr_gp_io_mux_spkr_ctl",
			    "csr_gp_io_lpaif_pri_pcm_pri_mode_muxsel",
			    "csr_gp_io_mux_quin_ctl";

		qcom,msm-ext-pa = "primary";
		qcom,msm-mclk-freq = <9600000>;		// 驱动编解码器的所需的时钟
		qcom,msm-mbhc-hphl-swh = <0>;
		qcom,msm-mbhc-gnd-swh = <0>;
		qcom,msm-hs-micbias-type = "external";
		qcom,msm-micbias1-ext-cap;		// 听筒具有外部电容
		qcom,audio-routing =
			"RX_BIAS", "MCLK",
			"SPK_RX_BIAS", "MCLK",
			"INT_LDO_H", "MCLK",
			"MIC BIAS External", "Handset Mic",
			"MIC BIAS External2", "Headset Mic",
			"MIC BIAS External", "Secondary Mic",
			"AMIC1", "MIC BIAS External",
			"AMIC2", "MIC BIAS External2",
			"AMIC3", "MIC BIAS External",
			"WSA_SPK OUT", "VDD_WSA_SWITCH",
			"SpkrMono WSA_IN", "WSA_SPK OUT";	// 扬声器

		qcom,hdmi-dba-codec-rx;

		qcom,msm-gpios =
			"pri_i2s",
			"us_eu_gpio",
			"quin_i2s";
		qcom,pinctrl-names =
			"all_off",
			"pri_i2s_act",
			"us_eu_gpio_act",
			"pri_i2s_us_eu_gpio_act",
			"quin_act",
			"quin_pri_i2s_act",
			"quin_us_eu_gpio_act",
			"quin_us_eu_gpio_pri_i2s_act";
		// 8种功能，不同的功能使用的引脚见pinctrl-x
		pinctrl-names =
			"all_off",
			"pri_i2s_act",
			"us_eu_gpio_act",
			"pri_i2s_us_eu_gpio_act",
			"quin_act",
			"quin_pri_i2s_act",
			"quin_us_eu_gpio_act",
			"quin_us_eu_gpio_pri_i2s_act";
		// 对应all_off功能, 使用3组pin,总共7个引脚
		pinctrl-0 = <&cdc_pdm_lines_sus
				&cdc_pdm_lines_2_sus &cross_conn_det_sus>;
		// pri_i2s_act, 7个引脚
		pinctrl-1 = <&cdc_pdm_lines_act
				&cdc_pdm_lines_2_act &cross_conn_det_sus>;
		// "us_eu_gpio_act",
		pinctrl-2 = <&cdc_pdm_lines_sus
				&cdc_pdm_lines_2_sus &cross_conn_det_act>;
		// "pri_i2s_us_eu_gpio_act",
		pinctrl-3 = <&cdc_pdm_lines_act
				&cdc_pdm_lines_2_act &cross_conn_det_act>;
		// "quin_act",
		pinctrl-4 = <&cdc_pdm_lines_sus
				&cdc_pdm_lines_2_sus &cross_conn_det_sus>;
		// "quin_pri_i2s_act",
		pinctrl-5 = <&cdc_pdm_lines_act
				&cdc_pdm_lines_2_act &cross_conn_det_sus>;
		// "quin_us_eu_gpio_act",
		pinctrl-6 = <&cdc_pdm_lines_sus
				&cdc_pdm_lines_2_sus &cross_conn_det_act>;
		// "quin_us_eu_gpio_pri_i2s_act";
		pinctrl-7 = <&cdc_pdm_lines_act
				&cdc_pdm_lines_2_act &cross_conn_det_act>;

		asoc-platform = <&pcm0>, <&pcm1>, <&pcm2>, <&voip>, <&voice>,
				<&loopback>, <&compress>, <&hostless>,
				<&afe>, <&lsm>, <&routing>, <&lpa>;
		asoc-platform-names = "msm-pcm-dsp.0", "msm-pcm-dsp.1",
				"msm-pcm-dsp.2", "msm-voip-dsp",
				"msm-pcm-voice", "msm-pcm-loopback",
				"msm-compress-dsp", "msm-pcm-hostless",
				"msm-pcm-afe", "msm-lsm-client",
				"msm-pcm-routing", "msm-pcm-lpa";
		asoc-cpu = <&dai_pri_auxpcm>,
				<&dai_mi2s0>, <&dai_mi2s1>,
				<&dai_mi2s2>, <&dai_mi2s3>,
				<&dai_mi2s5>, <&dai_mi2s6>,
				<&sb_0_rx>, <&sb_0_tx>, <&sb_1_rx>, <&sb_1_tx>,
				<&sb_3_rx>, <&sb_3_tx>, <&sb_4_rx>, <&sb_4_tx>,
				<&bt_sco_rx>, <&bt_sco_tx>,
				<&int_fm_rx>, <&int_fm_tx>,
				<&afe_pcm_rx>, <&afe_pcm_tx>,
				<&afe_proxy_rx>, <&afe_proxy_tx>,
				<&incall_record_rx>, <&incall_record_tx>,
				<&incall_music_rx>, <&incall_music_2_rx>;
		asoc-cpu-names = "msm-dai-q6-auxpcm.1",
				"msm-dai-q6-mi2s.0", "msm-dai-q6-mi2s.1",
				"msm-dai-q6-mi2s.2", "msm-dai-q6-mi2s.3",
				"msm-dai-q6-mi2s.5", "msm-dai-q6-mi2s.6",
				"msm-dai-q6-dev.16384", "msm-dai-q6-dev.16385",
				"msm-dai-q6-dev.16386", "msm-dai-q6-dev.16387",
				"msm-dai-q6-dev.16390", "msm-dai-q6-dev.16391",
				"msm-dai-q6-dev.16392", "msm-dai-q6-dev.16393",
				"msm-dai-q6-dev.12288", "msm-dai-q6-dev.12289",
				"msm-dai-q6-dev.12292", "msm-dai-q6-dev.12293",
				"msm-dai-q6-dev.224", "msm-dai-q6-dev.225",
				"msm-dai-q6-dev.241", "msm-dai-q6-dev.240",
				"msm-dai-q6-dev.32771", "msm-dai-q6-dev.32772",
				"msm-dai-q6-dev.32773", "msm-dai-q6-dev.32770";
	};

```

msm8953-pinctrl.dtsi


```
		cdc_reset_ctrl {
			cdc_reset_sleep: cdc_reset_sleep {
				mux {
					pins = "gpio67";
					function = "gpio";
				};
				config {
					pins = "gpio67";
					drive-strength = <16>;
					bias-disable;
					output-low;
				};
			};
			cdc_reset_active:cdc_reset_active {
				mux {
					pins = "gpio67";
					function = "gpio";
				};
				config {
					pins = "gpio67";
					drive-strength = <16>;
					bias-pull-down;
					output-high;
				};
			};
		};

		cdc_mclk2_pin {
			cdc_mclk2_sleep: cdc_mclk2_sleep {
				mux {
					pins = "gpio66";
					function = "pri_mi2s";
				};
				config {
					pins = "gpio66";
					drive-strength = <2>; /* 2 mA */
					bias-pull-down;       /* PULL DOWN */
				};
			};
			cdc_mclk2_active: cdc_mclk2_active {
				mux {
					pins = "gpio66";
					function = "pri_mi2s";
				};
				config {
					pins = "gpio66";
					drive-strength = <8>; /* 8 mA */
					bias-disable;         /* NO PULL */
				};
			};
		};
		cdc-pdm-2-lines {
			cdc_pdm_lines_2_act: pdm_lines_2_on {
				mux {
					pins = "gpio70", "gpio71", "gpio72";
					function = "cdc_pdm0";
				};

				config {
					pins = "gpio70", "gpio71", "gpio72";
					drive-strength = <8>;
				};
			};

			cdc_pdm_lines_2_sus: pdm_lines_2_off {
				mux {
					pins = "gpio70", "gpio71", "gpio72";
					function = "cdc_pdm0";
				};

				config {
					pins = "gpio70", "gpio71", "gpio72";
					drive-strength = <2>;
					bias-disable;
				};
			};
		};

		cdc-pdm-lines {
			cdc_pdm_lines_act: pdm_lines_on {
				mux {
					pins = "gpio69", "gpio73", "gpio74";
					function = "cdc_pdm0";
				};

				config {
					pins = "gpio69", "gpio73", "gpio74";
					drive-strength = <8>;
				};
			};
			cdc_pdm_lines_sus: pdm_lines_off {
				mux {
					pins = "gpio69", "gpio73", "gpio74";
					function = "cdc_pdm0";
				};

				config {
					pins = "gpio69", "gpio73", "gpio74";
					drive-strength = <2>;
					bias-disable;
				};
			};
		};

		cdc-pdm-comp-lines {
			cdc_pdm_comp_lines_act: pdm_comp_lines_on {
				mux {
					pins = "gpio67", "gpio68";
					function = "cdc_pdm0";
				};

				config {
					pins = "gpio67", "gpio68";
					drive-strength = <8>;
				};
			};

			cdc_pdm_comp_lines_sus: pdm_comp_lines_off {
				mux {
					pins = "gpio67", "gpio68";
					function = "cdc_pdm0";
				};

				config {
					pins = "gpio67", "gpio68";
					drive-strength = <2>;
					bias-disable;
				};
			};
		};

		cross-conn-det {
			cross_conn_det_act: lines_on {
				mux {
					pins = "gpio63";
					function = "gpio";
				};

				config {
					pins = "gpio63";
					drive-strength = <8>;
					output-low;
					bias-pull-down;
				};
			};

			cross_conn_det_sus: lines_off {
				mux {
					pins = "gpio63";
					function = "gpio";
				};

				config {
					pins = "gpio63";
					drive-strength = <2>;
					bias-pull-down;
				};
			};
		};
```
msm-pm8953.dtsi

```
&spmi_bus {

	qcom,pm8953@0 {
		spmi-slave-container;
		reg = <0x0>;
		#address-cells = <1>;
		#size-cells = <1>;

		pm8953_revid: qcom,revid@100 {
			compatible = "qcom,qpnp-revid";
			reg = <0x100 0x100>;
		};

		qcom,power-on@800 {
			compatible = "qcom,qpnp-power-on";
			reg = <0x800 0x100>;
			interrupts = <0x0 0x8 0x0>,
				<0x0 0x8 0x1>,
				<0x0 0x8 0x4>,
				<0x0 0x8 0x5>;
			interrupt-names = "kpdpwr", "resin",
				"resin-bark", "kpdpwr-resin-bark";
			qcom,pon-dbc-delay = <15625>;
			qcom,system-reset;

			qcom,pon_1 {
				qcom,pon-type = <0>;
                                qcom,support-reset = <1>;
                                qcom,s1-timer = <6720>;//<10256>;
                                qcom,s2-timer = <2000>;
                                qcom,s2-type = <7>;
                                //default 1 warm reset,4 is shutdown,7 is hard reset
				qcom,pull-up = <1>;
				linux,code = <116>;
			};

			qcom,pon_2 {
				qcom,pon-type = <1>;
				qcom,pull-up = <1>;
				linux,code = <114>;
			};
		};

		pm8953_temp_alarm: qcom,temp-alarm@2400 {
			compatible = "qcom,qpnp-temp-alarm";
			reg = <0x2400 0x100>;
			interrupts = <0x0 0x24 0x0>;
			label = "pm8953_tz";
			qcom,channel-num = <8>;
			qcom,threshold-set = <0>;
			qcom,temp_alarm-vadc = <&pm8953_vadc>;
		};

		pm8953_coincell: qcom,coincell@2800 {
			compatible = "qcom,qpnp-coincell";
			reg = <0x2800 0x100>;
		};

		pm8953_mpps: mpps {
			compatible = "qcom,qpnp-pin";
			spmi-dev-container;
			gpio-controller;
			#gpio-cells = <2>;
			#address-cells = <1>;
			#size-cells = <1>;
			label = "pm8953-mpp";

			mpp@a000 {
				reg = <0xa000 0x100>;
				qcom,pin-num = <1>;
				status = "disabled";
			};

			mpp@a100 {
				reg = <0xa100 0x100>;
				qcom,pin-num = <2>;
			};

			mpp@a200 {
				reg = <0xa200 0x100>;
				qcom,pin-num = <3>;
				status = "disabled";
			};

			mpp@a300 {
				reg = <0xa300 0x100>;
				qcom,pin-num = <4>;
			};
		};

		pm8953_gpios: gpios {
			spmi-dev-container;
			compatible = "qcom,qpnp-pin";
			gpio-controller;
			#gpio-cells = <2>;
			#address-cells = <1>;
			#size-cells = <1>;
			label = "pm8953-gpio";

			gpio@c000 {
				reg = <0xc000 0x100>;
				qcom,pin-num = <1>;
				status = "disabled";
			};

			gpio@c100 {
				reg = <0xc100 0x100>;
				qcom,pin-num = <2>;
				status = "disabled";
			};

			gpio@c200 {
				reg = <0xc200 0x100>;
				qcom,pin-num = <3>;
				status = "disabled";
			};

			gpio@c300 {
				reg = <0xc300 0x100>;
				qcom,pin-num = <4>;
				status = "disabled";
			};

			gpio@c400 {
				reg = <0xc400 0x100>;
				qcom,pin-num = <5>;
				status = "disabled";
			};

			gpio@c500 {
				reg = <0xc500 0x100>;
				qcom,pin-num = <6>;
				status = "disabled";
			};

			gpio@c600 {
				reg = <0xc600 0x100>;
				qcom,pin-num = <7>;
				status = "disabled";
			};

			gpio@c700 {
				reg = <0xc700 0x100>;
				qcom,pin-num = <8>;
				status = "disabled";
			};
		};

		pm8953_vadc: vadc@3100 {
			compatible = "qcom,qpnp-vadc";
			reg = <0x3100 0x100>;
			#address-cells = <1>;
			#size-cells = <0>;
			interrupts = <0x0 0x31 0x0>;
			interrupt-names = "eoc-int-en-set";
			qcom,adc-bit-resolution = <15>;
			qcom,adc-vdd-reference = <1800>;
			qcom,vadc-poll-eoc;

			chan@8 {
				label = "die_temp";
				reg = <8>;
				qcom,decimation = <0>;
				qcom,pre-div-channel-scaling = <0>;
				qcom,calibration-type = "absolute";
				qcom,scale-function = <3>;
				qcom,hw-settle-time = <0>;
				qcom,fast-avg-setup = <0>;
			};

			chan@9 {
				label = "ref_625mv";
				reg = <9>;
				qcom,decimation = <0>;
				qcom,pre-div-channel-scaling = <0>;
				qcom,calibration-type = "absolute";
				qcom,scale-function = <0>;
				qcom,hw-settle-time = <0>;
				qcom,fast-avg-setup = <0>;
			};

			chan@a {
				label = "ref_1250v";
				reg = <0xa>;
				qcom,decimation = <0>;
				qcom,pre-div-channel-scaling = <0>;
				qcom,calibration-type = "absolute";
				qcom,scale-function = <0>;
				qcom,hw-settle-time = <0>;
				qcom,fast-avg-setup = <0>;
			};

			chan@c {
				label = "ref_buf_625mv";
				reg = <0xc>;
				qcom,decimation = <0>;
				qcom,pre-div-channel-scaling = <0>;
				qcom,calibration-type = "absolute";
				qcom,scale-function = <0>;
				qcom,hw-settle-time = <0>;
				qcom,fast-avg-setup = <0>;
			};
		};

		pm8953_adc_tm: vadc@3400 {
			compatible = "qcom,qpnp-adc-tm";
			reg = <0x3400 0x100>;
			#address-cells = <1>;
			#size-cells = <0>;
			interrupts =	<0x0 0x34 0x0>,
					<0x0 0x34 0x3>,
					<0x0 0x34 0x4>;
			interrupt-names =	"eoc-int-en-set",
						"high-thr-en-set",
						"low-thr-en-set";
			qcom,adc-bit-resolution = <15>;
			qcom,adc-vdd-reference = <1800>;
			qcom,adc_tm-vadc = <&pm8953_vadc>;

		};

		pm8953_rtc: qcom,pm8953_rtc {
			spmi-dev-container;
			compatible = "qcom,qpnp-rtc";
			#address-cells = <1>;
			#size-cells = <1>;
			qcom,qpnp-rtc-write = <0>;
			qcom,qpnp-rtc-alarm-pwrup = <0>;

			qcom,pm8953_rtc_rw@6000 {
				reg = <0x6000 0x100>;
			};

			qcom,pm8953_rtc_alarm@6100 {
				reg = <0x6100 0x100>;
				interrupts = <0x0 0x61 0x1>;
			};
		};

		pm8953_typec: qcom,pm8953_typec@bf00 {
			status = "disable";
			compatible = "qcom,qpnp-typec";
			reg = <0xbf00 0x100>;
			interrupts =    <0x0 0xbf 0x0>,
					<0x0 0xbf 0x1>,
					<0x0 0xbf 0x2>,
					<0x0 0xbf 0x3>,
					<0x0 0xbf 0x4>,
					<0x0 0xbf 0x6>,
					<0x0 0xbf 0x7>;

			interrupt-names =       "vrd-change",
						"ufp-detect",
						"ufp-detach",
						"dfp-detect",
						"dfp-detach",
						"vbus-err",
						"vconn-oc";
		};
	};

	pm8953_1: qcom,pm8953@1 {
		spmi-slave-container;
		reg = <0x1>;
		#address-cells = <1>;
		#size-cells = <1>;

		pm8953_pwm: pwm@bc00 {
			status = "disabled";
			compatible = "qcom,qpnp-pwm";
			reg = <0xbc00 0x100>;
			reg-names = "qpnp-lpg-channel-base";
			qcom,channel-id = <0>;
			qcom,supported-sizes = <6>, <9>;
			#pwm-cells = <2>;
		};
	};
};
```


Liu Tao 

2019-3-5