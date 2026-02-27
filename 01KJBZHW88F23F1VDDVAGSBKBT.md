---
title: 20241025 - 模仿DECIMER, 微调模型
confluence_page_id: 3342812
created_at: 2024-10-25T11:13:27+00:00
updated_at: 2024-10-29T07:31:01+00:00
---

# e目标

模仿decimer的思路, 处理 化学分子式图片的识别任务

  1. 用空的image encoder, 微调一个Qwen2的tranformer
  2. 将decimer的encoder迁移过来, 微调一个Qwen2的transfomer
  3. 微调偶一个 encoder + transformer结构 (是否必须?)

另外的路线: 直接微调Qwen2-VL-7B

# 目标1: 用空的image encoder, 微调一个Qwen2的tranformer

在llama-factory中, 实现一个plugin, 将image处理成一个常量: 

```
DECIMER_IMAGE_SEQLEN = 512
class Qwen2DecimerPlugin(BasePlugin):
    @override
    def process_token_ids(
        self,
        input_ids: List[int],
        labels: Optional[List[int]],
        images: Sequence["ImageInput"],
        videos: Sequence["VideoInput"],
        tokenizer: "PreTrainedTokenizer",
        processor: Optional["ProcessorMixin"],
    ) -> Tuple[List[int], Optional[List[int]]]:
        self._validate_input(images, videos)
        num_images = len(images)
        image_seqlen = num_images * DECIMER_IMAGE_SEQLEN
        image_token_id = tokenizer.convert_tokens_to_ids(self.image_token)
        input_ids = [image_token_id] * image_seqlen + input_ids
        if labels is not None:
            labels = [IGNORE_INDEX] * image_seqlen + labels

        return input_ids, labels

``` 

修改部分代码, 以增加template和mm_plugin: [llamafactory.zip](/assets/01KJBZHW88F23F1VDDVAGSBKBT/llamafactory.zip)

微调命令: 

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=1 \
llamafactory-cli train \
    --stage sft \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-7B \
    --preprocessing_num_workers 1 \
    --finetuning_type lora \
    --template qwen2_decimer \
    --flash_attn auto \
    --dataset_dir data \
    --dataset decimer \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 3.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 2 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2-7B/lora/train/decimer \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --val_size 0.1 \
    --eval_strategy steps \
    --eval_steps 100 \
    --per_device_eval_batch_size 1" > train.log 2>&1 &
``` 

效果: loss收敛到0.7后不再下降

```
{'loss': 2.5523, 'grad_norm': 8.929716110229492, 'learning_rate': 4.999982664510298e-05, 'epoch': 0.0, 'num_input_tokens_seen': 15904}
{'loss': 1.7787, 'grad_norm': 3.6519806385040283, 'learning_rate': 4.9999306582816073e-05, 'epoch': 0.01, 'num_input_tokens_seen': 31728}
{'loss': 1.446, 'grad_norm': 3.1820781230926514, 'learning_rate': 4.99984398203517e-05, 'epoch': 0.01, 'num_input_tokens_seen': 47504}
{'loss': 1.3626, 'grad_norm': 2.5270040035247803, 'learning_rate': 4.9997226369730465e-05, 'epoch': 0.01, 'num_input_tokens_seen': 63312}
{'loss': 1.3274, 'grad_norm': 1.598885416984558, 'learning_rate': 4.999566624778099e-05, 'epoch': 0.02, 'num_input_tokens_seen': 79088}
{'loss': 1.1894, 'grad_norm': 2.9164340496063232, 'learning_rate': 4.999375947613963e-05, 'epoch': 0.02, 'num_input_tokens_seen': 94896}
{'loss': 1.1907, 'grad_norm': 2.9032044410705566, 'learning_rate': 4.9991506081250274e-05, 'epoch': 0.02, 'num_input_tokens_seen': 110672}
{'loss': 1.1488, 'grad_norm': 3.0592315196990967, 'learning_rate': 4.9988906094363856e-05, 'epoch': 0.03, 'num_input_tokens_seen': 126400}
{'loss': 1.0918, 'grad_norm': 2.0206573009490967, 'learning_rate': 4.998595955153803e-05, 'epoch': 0.03, 'num_input_tokens_seen': 142256}
{'loss': 1.098, 'grad_norm': 1.9715527296066284, 'learning_rate': 4.99826664936366e-05, 'epoch': 0.04, 'num_input_tokens_seen': 158160}
{'loss': 1.0521, 'grad_norm': 2.028524875640869, 'learning_rate': 4.997902696632899e-05, 'epoch': 0.04, 'num_input_tokens_seen': 174064}
{'loss': 1.0308, 'grad_norm': 2.37412691116333, 'learning_rate': 4.9975041020089576e-05, 'epoch': 0.04, 'num_input_tokens_seen': 189840}
{'loss': 1.0896, 'grad_norm': 2.7885451316833496, 'learning_rate': 4.997070871019703e-05, 'epoch': 0.05, 'num_input_tokens_seen': 205568}
{'loss': 1.033, 'grad_norm': 5.284908294677734, 'learning_rate': 4.996603009673353e-05, 'epoch': 0.05, 'num_input_tokens_seen': 221392}
{'loss': 0.9556, 'grad_norm': 1.888021469116211, 'learning_rate': 4.996100524458391e-05, 'epoch': 0.05, 'num_input_tokens_seen': 237120}
{'loss': 0.9726, 'grad_norm': 3.1275832653045654, 'learning_rate': 4.99556342234348e-05, 'epoch': 0.06, 'num_input_tokens_seen': 252896}
{'loss': 0.9789, 'grad_norm': 2.446869373321533, 'learning_rate': 4.9949917107773613e-05, 'epoch': 0.06, 'num_input_tokens_seen': 268544}
{'loss': 0.9935, 'grad_norm': 2.2974531650543213, 'learning_rate': 4.9943853976887556e-05, 'epoch': 0.06, 'num_input_tokens_seen': 284496}
{'loss': 0.9807, 'grad_norm': 1.8737847805023193, 'learning_rate': 4.99374449148625e-05, 'epoch': 0.07, 'num_input_tokens_seen': 300176}
{'loss': 0.9384, 'grad_norm': 2.464878797531128, 'learning_rate': 4.993069001058182e-05, 'epoch': 0.07, 'num_input_tokens_seen': 316096}
{'loss': 0.9657, 'grad_norm': 2.619518995285034, 'learning_rate': 4.992358935772519e-05, 'epoch': 0.07, 'num_input_tokens_seen': 331952}
{'loss': 0.9304, 'grad_norm': 2.117633819580078, 'learning_rate': 4.991614305476724e-05, 'epoch': 0.08, 'num_input_tokens_seen': 347872}
{'loss': 0.9549, 'grad_norm': 2.343872308731079, 'learning_rate': 4.9908351204976224e-05, 'epoch': 0.08, 'num_input_tokens_seen': 363680}
{'loss': 0.9911, 'grad_norm': 2.503408670425415, 'learning_rate': 4.9900213916412544e-05, 'epoch': 0.09, 'num_input_tokens_seen': 379376}
{'loss': 0.9422, 'grad_norm': 2.4144937992095947, 'learning_rate': 4.989173130192733e-05, 'epoch': 0.09, 'num_input_tokens_seen': 395248}
{'loss': 0.9026, 'grad_norm': 2.1537652015686035, 'learning_rate': 4.988290347916079e-05, 'epoch': 0.09, 'num_input_tokens_seen': 411104}
{'loss': 0.8926, 'grad_norm': 2.564995527267456, 'learning_rate': 4.9873730570540636e-05, 'epoch': 0.1, 'num_input_tokens_seen': 426880}
{'loss': 0.9441, 'grad_norm': 2.823760986328125, 'learning_rate': 4.986421270328035e-05, 'epoch': 0.1, 'num_input_tokens_seen': 442640}
{'loss': 0.8984, 'grad_norm': 2.282989978790283, 'learning_rate': 4.985435000937746e-05, 'epoch': 0.1, 'num_input_tokens_seen': 458464}
{'loss': 0.8998, 'grad_norm': 2.340827703475952, 'learning_rate': 4.984414262561165e-05, 'epoch': 0.11, 'num_input_tokens_seen': 474320}
{'loss': 0.9232, 'grad_norm': 2.6789302825927734, 'learning_rate': 4.983359069354292e-05, 'epoch': 0.11, 'num_input_tokens_seen': 490016}
{'loss': 0.93, 'grad_norm': 2.3173844814300537, 'learning_rate': 4.982269435950961e-05, 'epoch': 0.11, 'num_input_tokens_seen': 505760}
{'loss': 0.9095, 'grad_norm': 2.7692854404449463, 'learning_rate': 4.9811453774626335e-05, 'epoch': 0.12, 'num_input_tokens_seen': 521520}
{'loss': 0.8899, 'grad_norm': 3.4919028282165527, 'learning_rate': 4.979986909478195e-05, 'epoch': 0.12, 'num_input_tokens_seen': 537376}
{'loss': 0.8472, 'grad_norm': 3.159658193588257, 'learning_rate': 4.978794048063731e-05, 'epoch': 0.12, 'num_input_tokens_seen': 553136}
{'loss': 0.8617, 'grad_norm': 3.396265983581543, 'learning_rate': 4.977566809762311e-05, 'epoch': 0.13, 'num_input_tokens_seen': 569056}
{'loss': 0.934, 'grad_norm': 2.813883066177368, 'learning_rate': 4.976305211593758e-05, 'epoch': 0.13, 'num_input_tokens_seen': 584928}
{'loss': 0.8874, 'grad_norm': 2.753955364227295, 'learning_rate': 4.975009271054409e-05, 'epoch': 0.14, 'num_input_tokens_seen': 600656}
{'loss': 0.8244, 'grad_norm': 2.9506993293762207, 'learning_rate': 4.9736790061168756e-05, 'epoch': 0.14, 'num_input_tokens_seen': 616368}
{'loss': 0.8962, 'grad_norm': 2.3765571117401123, 'learning_rate': 4.9723144352297935e-05, 'epoch': 0.14, 'num_input_tokens_seen': 632080}
{'loss': 0.8587, 'grad_norm': 2.470353603363037, 'learning_rate': 4.9709155773175644e-05, 'epoch': 0.15, 'num_input_tokens_seen': 647936}
{'loss': 0.9007, 'grad_norm': 2.784029960632324, 'learning_rate': 4.9694824517800995e-05, 'epoch': 0.15, 'num_input_tokens_seen': 663760}
{'loss': 0.8813, 'grad_norm': 2.944326162338257, 'learning_rate': 4.968015078492544e-05, 'epoch': 0.15, 'num_input_tokens_seen': 679456}
{'loss': 0.912, 'grad_norm': 2.2745604515075684, 'learning_rate': 4.966513477805007e-05, 'epoch': 0.16, 'num_input_tokens_seen': 695312}
{'loss': 0.8842, 'grad_norm': 2.546173095703125, 'learning_rate': 4.9649776705422735e-05, 'epoch': 0.16, 'num_input_tokens_seen': 710992}
{'loss': 0.8407, 'grad_norm': 2.685520887374878, 'learning_rate': 4.963407678003522e-05, 'epoch': 0.16, 'num_input_tokens_seen': 726832}
{'loss': 0.8902, 'grad_norm': 2.618696451187134, 'learning_rate': 4.961803521962022e-05, 'epoch': 0.17, 'num_input_tokens_seen': 742784}
{'loss': 0.8586, 'grad_norm': 2.5750796794891357, 'learning_rate': 4.96016522466484e-05, 'epoch': 0.17, 'num_input_tokens_seen': 758528}
{'loss': 0.8232, 'grad_norm': 3.1608352661132812, 'learning_rate': 4.958492808832523e-05, 'epoch': 0.17, 'num_input_tokens_seen': 774336}
{'loss': 0.8523, 'grad_norm': 2.98606014251709, 'learning_rate': 4.956786297658791e-05, 'epoch': 0.18, 'num_input_tokens_seen': 790160}
{'loss': 0.8057, 'grad_norm': 2.5154671669006348, 'learning_rate': 4.955045714810207e-05, 'epoch': 0.18, 'num_input_tokens_seen': 806096}
{'loss': 0.7885, 'grad_norm': 2.6993985176086426, 'learning_rate': 4.953271084425858e-05, 'epoch': 0.18, 'num_input_tokens_seen': 821904}
{'loss': 0.8384, 'grad_norm': 2.8095574378967285, 'learning_rate': 4.9514624311170124e-05, 'epoch': 0.19, 'num_input_tokens_seen': 837728}
{'loss': 0.8572, 'grad_norm': 2.538350820541382, 'learning_rate': 4.9496197799667835e-05, 'epoch': 0.19, 'num_input_tokens_seen': 853632}
{'loss': 0.887, 'grad_norm': 2.551886796951294, 'learning_rate': 4.9477431565297774e-05, 'epoch': 0.2, 'num_input_tokens_seen': 869296}
{'loss': 0.8255, 'grad_norm': 2.732753276824951, 'learning_rate': 4.945832586831746e-05, 'epoch': 0.2, 'num_input_tokens_seen': 885104}
{'loss': 0.8287, 'grad_norm': 3.2146453857421875, 'learning_rate': 4.943888097369216e-05, 'epoch': 0.2, 'num_input_tokens_seen': 900896}
{'loss': 0.8639, 'grad_norm': 2.9741921424865723, 'learning_rate': 4.94190971510913e-05, 'epoch': 0.21, 'num_input_tokens_seen': 916720}
{'loss': 0.8271, 'grad_norm': 2.557722806930542, 'learning_rate': 4.9398974674884676e-05, 'epoch': 0.21, 'num_input_tokens_seen': 932656}
{'loss': 0.8597, 'grad_norm': 2.4983863830566406, 'learning_rate': 4.937851382413869e-05, 'epoch': 0.21, 'num_input_tokens_seen': 948480}
{'loss': 0.8375, 'grad_norm': 2.568315267562866, 'learning_rate': 4.935771488261242e-05, 'epoch': 0.22, 'num_input_tokens_seen': 964320}
{'loss': 0.8172, 'grad_norm': 2.909195899963379, 'learning_rate': 4.933657813875373e-05, 'epoch': 0.22, 'num_input_tokens_seen': 980208}
{'loss': 0.8509, 'grad_norm': 2.828169822692871, 'learning_rate': 4.931510388569528e-05, 'epoch': 0.22, 'num_input_tokens_seen': 996000}
{'loss': 0.8058, 'grad_norm': 2.7635133266448975, 'learning_rate': 4.929329242125041e-05, 'epoch': 0.23, 'num_input_tokens_seen': 1011744}
{'loss': 0.7985, 'grad_norm': 2.4780004024505615, 'learning_rate': 4.927114404790907e-05, 'epoch': 0.23, 'num_input_tokens_seen': 1027584}
{'loss': 0.7813, 'grad_norm': 3.075150728225708, 'learning_rate': 4.924865907283356e-05, 'epoch': 0.23, 'num_input_tokens_seen': 1043408}
{'loss': 0.8232, 'grad_norm': 2.455781936645508, 'learning_rate': 4.922583780785434e-05, 'epoch': 0.24, 'num_input_tokens_seen': 1059168}
{'loss': 0.7776, 'grad_norm': 2.913130283355713, 'learning_rate': 4.9202680569465645e-05, 'epoch': 0.24, 'num_input_tokens_seen': 1074944}
{'loss': 0.8367, 'grad_norm': 2.617614984512329, 'learning_rate': 4.9179187678821124e-05, 'epoch': 0.25, 'num_input_tokens_seen': 1090800}
{'loss': 0.8357, 'grad_norm': 2.817324161529541, 'learning_rate': 4.91553594617294e-05, 'epoch': 0.25, 'num_input_tokens_seen': 1106688}
{'loss': 0.8089, 'grad_norm': 2.6203839778900146, 'learning_rate': 4.91311962486495e-05, 'epoch': 0.25, 'num_input_tokens_seen': 1122480}
{'loss': 0.8272, 'grad_norm': 2.6568124294281006, 'learning_rate': 4.910669837468637e-05, 'epoch': 0.26, 'num_input_tokens_seen': 1138272}
{'loss': 0.806, 'grad_norm': 2.5884957313537598, 'learning_rate': 4.908186617958608e-05, 'epoch': 0.26, 'num_input_tokens_seen': 1154048}
{'loss': 0.8091, 'grad_norm': 2.8519089221954346, 'learning_rate': 4.905670000773126e-05, 'epoch': 0.26, 'num_input_tokens_seen': 1169872}
{'loss': 0.767, 'grad_norm': 2.6895828247070312, 'learning_rate': 4.903120020813625e-05, 'epoch': 0.27, 'num_input_tokens_seen': 1185824}
{'loss': 0.8106, 'grad_norm': 2.5748798847198486, 'learning_rate': 4.9005367134442235e-05, 'epoch': 0.27, 'num_input_tokens_seen': 1201664}
{'loss': 0.8028, 'grad_norm': 3.2030317783355713, 'learning_rate': 4.8979201144912426e-05, 'epoch': 0.27, 'num_input_tokens_seen': 1217440}
{'loss': 0.8221, 'grad_norm': 3.1476728916168213, 'learning_rate': 4.895270260242701e-05, 'epoch': 0.28, 'num_input_tokens_seen': 1233104}
{'loss': 0.8141, 'grad_norm': 2.4444470405578613, 'learning_rate': 4.892587187447815e-05, 'epoch': 0.28, 'num_input_tokens_seen': 1249104}
{'loss': 0.8113, 'grad_norm': 2.7521164417266846, 'learning_rate': 4.889870933316488e-05, 'epoch': 0.28, 'num_input_tokens_seen': 1264912}
{'loss': 0.7876, 'grad_norm': 2.730365037918091, 'learning_rate': 4.8871215355187994e-05, 'epoch': 0.29, 'num_input_tokens_seen': 1280656}
{'loss': 0.8239, 'grad_norm': 3.1310253143310547, 'learning_rate': 4.884339032184474e-05, 'epoch': 0.29, 'num_input_tokens_seen': 1296512}
{'loss': 0.8085, 'grad_norm': 2.652570962905884, 'learning_rate': 4.881523461902356e-05, 'epoch': 0.3, 'num_input_tokens_seen': 1312368}
{'loss': 0.7716, 'grad_norm': 2.6185295581817627, 'learning_rate': 4.878674863719879e-05, 'epoch': 0.3, 'num_input_tokens_seen': 1328272}
{'loss': 0.7988, 'grad_norm': 2.7328407764434814, 'learning_rate': 4.8757932771425184e-05, 'epoch': 0.3, 'num_input_tokens_seen': 1344064}
{'loss': 0.808, 'grad_norm': 2.8240792751312256, 'learning_rate': 4.872878742133246e-05, 'epoch': 0.31, 'num_input_tokens_seen': 1360000}
{'loss': 0.7951, 'grad_norm': 2.588547945022583, 'learning_rate': 4.8699312991119746e-05, 'epoch': 0.31, 'num_input_tokens_seen': 1375904}
{'loss': 0.7995, 'grad_norm': 2.9922497272491455, 'learning_rate': 4.8669509889549986e-05, 'epoch': 0.31, 'num_input_tokens_seen': 1391664}
{'loss': 0.775, 'grad_norm': 2.9460060596466064, 'learning_rate': 4.8639378529944266e-05, 'epoch': 0.32, 'num_input_tokens_seen': 1407488}
{'loss': 0.8118, 'grad_norm': 2.6677591800689697, 'learning_rate': 4.860891933017609e-05, 'epoch': 0.32, 'num_input_tokens_seen': 1423360}
{'loss': 0.7363, 'grad_norm': 2.6442394256591797, 'learning_rate': 4.857813271266558e-05, 'epoch': 0.32, 'num_input_tokens_seen': 1439120}
{'loss': 0.8165, 'grad_norm': 2.80317759513855, 'learning_rate': 4.85470191043736e-05, 'epoch': 0.33, 'num_input_tokens_seen': 1454928}
{'loss': 0.7911, 'grad_norm': 2.955376625061035, 'learning_rate': 4.851557893679586e-05, 'epoch': 0.33, 'num_input_tokens_seen': 1470768}
{'loss': 0.8204, 'grad_norm': 2.4911983013153076, 'learning_rate': 4.8483812645956925e-05, 'epoch': 0.33, 'num_input_tokens_seen': 1486688}
{'loss': 0.8105, 'grad_norm': 2.7746458053588867, 'learning_rate': 4.845172067240415e-05, 'epoch': 0.34, 'num_input_tokens_seen': 1502496}
{'loss': 0.791, 'grad_norm': 2.8079679012298584, 'learning_rate': 4.841930346120161e-05, 'epoch': 0.34, 'num_input_tokens_seen': 1518256}
{'loss': 0.7556, 'grad_norm': 3.2076504230499268, 'learning_rate': 4.838656146192388e-05, 'epoch': 0.34, 'num_input_tokens_seen': 1534160}
{'loss': 0.7521, 'grad_norm': 3.1094963550567627, 'learning_rate': 4.8353495128649834e-05, 'epoch': 0.35, 'num_input_tokens_seen': 1549936}
{'loss': 0.7527, 'grad_norm': 2.8089849948883057, 'learning_rate': 4.832010491995634e-05, 'epoch': 0.35, 'num_input_tokens_seen': 1565904}
{'loss': 0.7742, 'grad_norm': 2.4083590507507324, 'learning_rate': 4.828639129891189e-05, 'epoch': 0.36, 'num_input_tokens_seen': 1581712}
{'loss': 0.7575, 'grad_norm': 2.8905277252197266, 'learning_rate': 4.825235473307021e-05, 'epoch': 0.36, 'num_input_tokens_seen': 1597600}
{'loss': 0.759, 'grad_norm': 2.705479860305786, 'learning_rate': 4.821799569446368e-05, 'epoch': 0.36, 'num_input_tokens_seen': 1613376}
{'loss': 0.7372, 'grad_norm': 2.822953939437866, 'learning_rate': 4.818331465959696e-05, 'epoch': 0.37, 'num_input_tokens_seen': 1629248}
{'loss': 0.7326, 'grad_norm': 3.0343849658966064, 'learning_rate': 4.8148312109440195e-05, 'epoch': 0.37, 'num_input_tokens_seen': 1645040}
{'loss': 0.7533, 'grad_norm': 2.794076919555664, 'learning_rate': 4.811298852942248e-05, 'epoch': 0.37, 'num_input_tokens_seen': 1660912}
{'loss': 0.779, 'grad_norm': 2.5467209815979004, 'learning_rate': 4.807734440942505e-05, 'epoch': 0.38, 'num_input_tokens_seen': 1676720}
{'loss': 0.8139, 'grad_norm': 2.684506893157959, 'learning_rate': 4.804138024377454e-05, 'epoch': 0.38, 'num_input_tokens_seen': 1692688}
{'loss': 0.7436, 'grad_norm': 2.72198224067688, 'learning_rate': 4.8005096531236074e-05, 'epoch': 0.38, 'num_input_tokens_seen': 1708512}
{'loss': 0.736, 'grad_norm': 2.554854393005371, 'learning_rate': 4.7968493775006396e-05, 'epoch': 0.39, 'num_input_tokens_seen': 1724320}
{'loss': 0.7218, 'grad_norm': 2.41042423248291, 'learning_rate': 4.793157248270687e-05, 'epoch': 0.39, 'num_input_tokens_seen': 1740144}
{'loss': 0.7504, 'grad_norm': 2.944502830505371, 'learning_rate': 4.789433316637644e-05, 'epoch': 0.39, 'num_input_tokens_seen': 1755936}
{'loss': 0.7498, 'grad_norm': 3.13153076171875, 'learning_rate': 4.7856776342464535e-05, 'epoch': 0.4, 'num_input_tokens_seen': 1771664}
{'loss': 0.8132, 'grad_norm': 3.0339887142181396, 'learning_rate': 4.78189025318239e-05, 'epoch': 0.4, 'num_input_tokens_seen': 1787472}
{'loss': 0.6957, 'grad_norm': 2.723206043243408, 'learning_rate': 4.77807122597034e-05, 'epoch': 0.41, 'num_input_tokens_seen': 1803264}
{'loss': 0.7559, 'grad_norm': 2.6813106536865234, 'learning_rate': 4.774220605574066e-05, 'epoch': 0.41, 'num_input_tokens_seen': 1819056}
{'loss': 0.7342, 'grad_norm': 2.978971004486084, 'learning_rate': 4.770338445395481e-05, 'epoch': 0.41, 'num_input_tokens_seen': 1834816}
{'loss': 0.7923, 'grad_norm': 2.8708574771881104, 'learning_rate': 4.766424799273904e-05, 'epoch': 0.42, 'num_input_tokens_seen': 1850672}
{'loss': 0.7786, 'grad_norm': 2.7228827476501465, 'learning_rate': 4.7624797214853116e-05, 'epoch': 0.42, 'num_input_tokens_seen': 1866400}
{'loss': 0.7175, 'grad_norm': 2.598132848739624, 'learning_rate': 4.75850326674159e-05, 'epoch': 0.42, 'num_input_tokens_seen': 1882192}
{'loss': 0.7537, 'grad_norm': 3.0342555046081543, 'learning_rate': 4.7544954901897684e-05, 'epoch': 0.43, 'num_input_tokens_seen': 1897984}
``` 

# 目标2: 将decimer的encoder迁移过来

参考代码: DECIMER_Predictor:

```
encoder的初始化逻辑
    encoder = Efficient_Net_encoder.Encoder(**encoder_config)

encoder的初始化逻辑和加载逻辑
    optimizer, encoder, transformer = config.prepare_models(
        encoder_config=testing_config.encoder_config,
        transformer_config=testing_config.transformer_config,
        replica_batch_size=REPLICA_BATCH_SIZE,
        verbose=0,
    )

    # Load trained model checkpoint
    checkpoint_path = "checkpoints"
    ckpt = tf.train.Checkpoint(
        encoder=encoder, transformer=transformer, optimizer=optimizer
    )
    ckpt_manager = tf.train.CheckpointManager(ckpt, checkpoint_path, max_to_keep=50)
    if ckpt_manager.latest_checkpoint:
        ckpt.restore(tf.train.latest_checkpoint(checkpoint_path))
        start_epoch = int(ckpt_manager.latest_checkpoint.split("-")[-1])
 
DECIMER_Predictor的调用逻辑
        _image_batch = tf.expand_dims(Decoded_image, 0)
        _image_embedding = encoder(_image_batch, training=False)

保存逻辑??
tf.saved_model.save(
    DECIMER_export,
    export_dir="DECIMER_Packed_model",
    options=tf.saved_model.SaveOptions(experimental_custom_gradients=True),
)

``` 

# 微调Qwen2-VL-7B

先用1k数据尝试: 

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=1 HTTP_PROXY=http://10.186.16.136:7890 HTTPS_PROXY=http://10.186.16.136:7890 CUDA_LAUNCH_BLOCKING=1 TF_ENABLE_ONEDNN_OPTS=0 \
llamafactory-cli train \
    --stage sft \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-VL-7B-Instruct \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template qwen2_vl \
    --flash_attn auto \
    --dataset_dir data \
    --dataset decimer-1000 \
    --cutoff_len 1024 \
    --learning_rate 5e-05 \
    --num_train_epochs 10.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 2 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0 \
    --lora_target all \
    --val_size 0.1 \
    --eval_strategy steps \
    --eval_steps 100 \
    --per_device_eval_batch_size 1" > train.log 2>&1 &
``` 

发现过拟合

![image2024-10-28 14:47:8.png](/assets/01KJBZHW88F23F1VDDVAGSBKBT/image2024-10-28%2014%3A47%3A8.png)![image2024-10-28 14:47:11.png](/assets/01KJBZHW88F23F1VDDVAGSBKBT/image2024-10-28%2014%3A47%3A11.png)

调整参数, 并降低epoch: 

  - \--learning_rate 1e-5
  - \--lora_dropout 0.1
  - \--per_device_train_batch_size 4

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=1 HTTP_PROXY=http://10.186.16.136:7890 HTTPS_PROXY=http://10.186.16.136:7890 CUDA_LAUNCH_BLOCKING=1 TF_ENABLE_ONEDNN_OPTS=0 \
llamafactory-cli train \
    --stage sft \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-VL-7B-Instruct \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template qwen2_vl \
    --flash_attn auto \
    --dataset_dir data \
    --dataset decimer-1000 \
    --cutoff_len 1024 \
    --learning_rate 1e-05 \
    --num_train_epochs 6.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 4 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0.1 \
    --lora_target all \
    --val_size 0.1 \
    --eval_strategy steps \
    --eval_steps 50 \
    --per_device_eval_batch_size 2" > train.log 2>&1 &
``` 

效果: 没有出现过拟合

![image2024-10-28 23:15:52.png](/assets/01KJBZHW88F23F1VDDVAGSBKBT/image2024-10-28%2023%3A15%3A52.png)![image2024-10-28 23:16:0.png](/assets/01KJBZHW88F23F1VDDVAGSBKBT/image2024-10-28%2023%3A16%3A0.png)

计划: 

  1. 使用4090, 搭建环境, 进行更大数据规模的训练
  2. 在以上环境中, 将Image -> SMILES的训练数据, 转换为 Image -> IUPAC

# 训练Image -> IUPAC

对于前1000个数据, 使用脚本, 通过URL: "<https://cactus.nci.nih.gov/chemical/structure/CCOC(=O)c1c(NC2=NS(=O)(=O)c3ccccc32)sc2c1CCC(C)C2/iupac_name>" 来获取IUPAC name. 

有效数据为 793个, 其他207个数据无法通过SMILES表达式找到IUPAC name

使用这793个数据进行训练, 参数与上面相同 (调低了per_device_train_batch_size, 否则OOM)

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=1 HTTP_PROXY=http://10.186.16.136:7890 HTTPS_PROXY=http://10.186.16.136:7890 CUDA_LAUNCH_BLOCKING=1 TF_ENABLE_ONEDNN_OPTS=0 \
llamafactory-cli train \
    --stage sft \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-VL-7B-Instruct \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template qwen2_vl \
    --flash_attn auto \
    --dataset_dir data \
    --dataset decimer-img2iupac-1000 \
    --cutoff_len 1024 \
    --learning_rate 1e-05 \
    --num_train_epochs 6.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 2 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2iupac-1000 \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0.1 \
    --lora_target all \
    --val_size 0.1 \
    --eval_strategy steps \
    --eval_steps 50 \
    --per_device_eval_batch_size 2" > train.log 2>&1 &
``` 

![image2024-10-29 10:2:31.png](/assets/01KJBZHW88F23F1VDDVAGSBKBT/image2024-10-29%2010%3A2%3A31.png)![image2024-10-29 10:2:23.png](/assets/01KJBZHW88F23F1VDDVAGSBKBT/image2024-10-29%2010%3A2%3A23.png)

eval_loss的下降, 不如 Img->smiles 好

# 训练Image -> stdinchi

类似于上一节, 拿到stdinchi

```
nohup bash -c "PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True CUDA_VISIBLE_DEVICES=1 HTTP_PROXY=http://10.186.16.136:7890 HTTPS_PROXY=http://10.186.16.136:7890 CUDA_LAUNCH_BLOCKING=1 TF_ENABLE_ONEDNN_OPTS=0 \
llamafactory-cli train \
    --stage sft \
    --do_train True \
    --model_name_or_path Qwen/Qwen2-VL-7B-Instruct \
    --preprocessing_num_workers 16 \
    --finetuning_type lora \
    --template qwen2_vl \
    --flash_attn auto \
    --dataset_dir data \
    --dataset decimer-img2stdinchi-1000 \
    --cutoff_len 1024 \
    --learning_rate 1e-05 \
    --num_train_epochs 6.0 \
    --max_samples 100000 \
    --per_device_train_batch_size 2 \
    --gradient_accumulation_steps 8 \
    --lr_scheduler_type cosine \
    --max_grad_norm 1.0 \
    --logging_steps 5 \
    --save_steps 100 \
    --warmup_steps 0 \
    --optim adamw_torch \
    --packing False \
    --report_to none \
    --output_dir saves/Qwen2VL-7B-Chat/lora/train_qwen2vl_decimer-img2stdinchi-1000 \
    --bf16 True \
    --plot_loss True \
    --ddp_timeout 180000000 \
    --include_num_input_tokens_seen True \
    --lora_rank 8 \
    --lora_alpha 16 \
    --lora_dropout 0.1 \
    --lora_target all \
    --val_size 0.1 \
    --eval_strategy steps \
    --eval_steps 50 \
    --per_device_eval_batch_size 2" > train.log 2>&1 &
``` 

![image2024-10-29 15:32:21.png](/assets/01KJBZHW88F23F1VDDVAGSBKBT/image2024-10-29%2015%3A32%3A21.png)![image2024-10-29 15:32:26.png](/assets/01KJBZHW88F23F1VDDVAGSBKBT/image2024-10-29%2015%3A32%3A26.png)

eval_loss的下降, 不如 Img->smiles 好
