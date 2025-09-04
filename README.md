# Cosyvoice2.Axera
Cosyvoice2 语音生成demo on Axera

- clone 代码：git clone --recursive https://github.com/AXERA-TECH/Cosyvoice2.Axera.git
- 模型转换请参考[模型转换](/model_convert/README.md)
- [c++ demo](cpp/README.md)

## 支持平台

- [x] AX650N

## 模型转换

[模型转换](./model_convert/README.md)

## 上板部署

- AX650N 的设备已预装 Ubuntu22.04
- 以 root 权限登陆 AX650N 的板卡设备
- 链接互联网，确保 AX650N 的设备能正常执行 `apt install`, `pip install` 等指令
- 已验证设备：AX650N DEMO Board

## 运行示例  

### 1. 音频生成（音色复刻）  

#### 1. 安装python  
需要第2，3步需要使用这些python包，如果在PC上运行第2，3步，就在PC上安装。  
```
pip3 install -r cpp/scripts/requirements.txt
```  

#### 2. 首先处理prompt speech  
```
cd src
python ../scripts/process_prompt.py
```
注意根据实际情况传入参数
```
args.add_argument('--model_dir', type=str, default="../../model_convert/pretrained_models/CosyVoice2-0.5B/")
args.add_argument('--wetext_dir', type=str, default="../../model_convert/pengzhendong/wetext/")
args.add_argument('--sample_rate', type=int, default=24000)
args.add_argument('--zero_shot_spk_id', type=str, default="")
args.add_argument('--tts_text', type=str, default="君不见黄河之水天上来，奔流到海不复回。君不见高堂明镜悲白发，朝如青丝暮成雪。")
args.add_argument('--prompt_text', type=str, default="希望你以后能够做的比我还好呦。")
args.add_argument('--prompt_speech', type=str, default="../../model_convert/asset/zero_shot_prompt.wav")
```

#### 3. 启动 HTTP Tokenizer Server  
```
cd scripts
python cosyvoice2_tokenizer.py --host {your host} --port {your port}   # 和 run.sh 中一致
```

#### 4. 在板子上运行模型  
1) 先修改 `run.sh` 中的http host.  
2) 将 `scripts/run.sh`, `build/install/bin/main`, `process_prompt.py 生成的文件` 拷贝到爱芯板子上  
3) 运行 `run.sh`  
```shell
root@ax650 ~/yongqiang/lhj/Cosyvoice2.Axera/cpp/src # bash run.sh 
rm: cannot remove 'output*.wav': No such file or directory
[I][                            Init][ 108]: LLM init start
[I][                            Init][  34]: connect http://10.122.86.184:12345 ok
bos_id: 0, eos_id: 1773
  7% | ███                               |   2 /  27 [3.11s<42.04s, 0.64 count/s] embed_selector init ok[I][                            Init][ 138]: attr.axmodel_num:24
100% | ████████████████████████████████ |  27 /  27 [10.32s<10.32s, 2.62 count/s] init post axmodel ok,remain_cmm(7178 MB)
[I][                            Init][ 216]: max_token_len : 1023
[I][                            Init][ 221]: kv_cache_size : 128, kv_cache_num: 1023
[I][                            Init][ 229]: prefill_token_num : 128
[I][                            Init][ 233]: grp: 1, prefill_max_token_num : 1
[I][                            Init][ 233]: grp: 2, prefill_max_token_num : 128
[I][                            Init][ 233]: grp: 3, prefill_max_token_num : 256
[I][                            Init][ 233]: grp: 4, prefill_max_token_num : 384
[I][                            Init][ 233]: grp: 5, prefill_max_token_num : 512
[I][                            Init][ 237]: prefill_max_token_num : 512
[I][                            Init][ 249]: LLM init ok
[I][                            Init][ 154]: Token2Wav init ok
[I][                            main][ 273]: 
[I][                             Run][ 388]: input token num : 142, prefill_split_num : 2
[I][                             Run][ 422]: input_num_token:128
[I][                             Run][ 422]: input_num_token:14
[I][                             Run][ 607]: ttft: 236.90 ms
[Main/Token2Wav Thread] Processing batch of 28 tokens...
Successfully saved audio to output_0.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 53 tokens...
Successfully saved audio to output_1.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_2.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_3.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_4.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_5.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_6.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_7.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_8.wav (32-bit Float PCM).
[Main/Token2Wav Thread] Processing batch of 78 tokens...
Successfully saved audio to output_9.wav (32-bit Float PCM).
[I][                             Run][ 723]: hit eos, llm finished
[I][                             Run][ 753]: llm finished
[Main/Token2Wav Thread] Buffer is empty and LLM finished. Exiting.


[I][                             Run][ 758]: total decode tokens:271
[N][                             Run][ 759]: hit eos,avg 21.47 token/s

Successfully saved audio to output_10.wav (32-bit Float PCM).
Successfully saved audio to output.wav (32-bit Float PCM).

Voice generation pipeline completed.
Type "q" to exit, Ctrl+c to stop current running
text >> 
```
音频输出：
[output.wav](asset/output.wav)


##  音频生成速度  
| Stage | Time |
|------|------|
| llm prefill ( input_token_num + prompt_token_num 在 [0,128 ] ) | 104 ms  | 
| llm prefill ( input_token_num + prompt_token_num 在 [128,256 ] ) | 234 ms  | 
| llm Decode + Token2Wav  |  21.24 token/s |

## Reference

- [Cosyvoice](https://github.com/FunAudioLLM/CosyVoice)

## 技术讨论

- Github issues
- QQ 群: 139953715

## 常见问题 
[QA.md](QA.md)