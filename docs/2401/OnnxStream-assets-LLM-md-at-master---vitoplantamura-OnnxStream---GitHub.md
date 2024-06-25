<!--yml

分类：未分类

日期：2024年5月27日 14:46:39

-->

# [OnnxStream/assets/LLM.md](https://github.com/vitoplantamura/OnnxStream/blob/master/assets/LLM.md) 在 GitHub 上的 vitoplantamura/OnnxStream 仓库。

> 来源：[https://github.com/vitoplantamura/OnnxStream/blob/master/assets/LLM.md](https://github.com/vitoplantamura/OnnxStream/blob/master/assets/LLM.md)

```
import transformers 
import torch
import torch.nn as nn
import onnx
from transformers import AutoModelForCausalLM

model = AutoModelForCausalLM.from_pretrained("mistralai/Mistral-7B-Instruct-v0.2")

class LlamaModel(nn.Module):
    def __init__(self, model):
        super(LlamaModel, self).__init__()
        self.model = model
    def forward(self, input_ids, attention_mask, position_ids,
                pkv0, pkv1, pkv2, pkv3, pkv4, pkv5, pkv6, pkv7, pkv8, pkv9, pkv10,
                pkv11, pkv12, pkv13, pkv14, pkv15, pkv16, pkv17, pkv18, pkv19, pkv20,
                pkv21, pkv22, pkv23, pkv24, pkv25, pkv26, pkv27, pkv28, pkv29, pkv30,
                pkv31, pkv32, pkv33, pkv34, pkv35, pkv36, pkv37, pkv38, pkv39, pkv40,
                pkv41, pkv42, pkv43, pkv44, pkv45, pkv46, pkv47, pkv48, pkv49, pkv50,
                pkv51, pkv52, pkv53, pkv54, pkv55, pkv56, pkv57, pkv58, pkv59, pkv60,
                pkv61, pkv62, pkv63):
        past_key_values = (
            (pkv0, pkv1), (pkv2, pkv3), (pkv4, pkv5), (pkv6, pkv7), (pkv8, pkv9), (pkv10,
            pkv11), (pkv12, pkv13), (pkv14, pkv15), (pkv16, pkv17), (pkv18, pkv19), (pkv20,
            pkv21), (pkv22, pkv23), (pkv24, pkv25), (pkv26, pkv27), (pkv28, pkv29), (pkv30,
            pkv31), (pkv32, pkv33), (pkv34, pkv35), (pkv36, pkv37), (pkv38, pkv39), (pkv40,
            pkv41), (pkv42, pkv43), (pkv44, pkv45), (pkv46, pkv47), (pkv48, pkv49), (pkv50,
            pkv51), (pkv52, pkv53), (pkv54, pkv55), (pkv56, pkv57), (pkv58, pkv59), (pkv60,
            pkv61), (pkv62, pkv63))
        o = self.model(use_cache=True, return_dict=True,
            input_ids=input_ids, attention_mask=attention_mask, position_ids=position_ids, past_key_values=past_key_values)
        pkv = o.past_key_values
        return [o.logits,
               pkv[0][0], pkv[0][1], pkv[1][0], pkv[1][1], pkv[2][0], pkv[2][1], pkv[3][0], pkv[3][1], pkv[4][0], pkv[4][1],
               pkv[5][0], pkv[5][1], pkv[6][0], pkv[6][1], pkv[7][0], pkv[7][1], pkv[8][0], pkv[8][1], pkv[9][0], pkv[9][1],
               pkv[10][0], pkv[10][1], pkv[11][0], pkv[11][1], pkv[12][0], pkv[12][1], pkv[13][0], pkv[13][1], pkv[14][0], pkv[14][1],
               pkv[15][0], pkv[15][1], pkv[16][0], pkv[16][1], pkv[17][0], pkv[17][1], pkv[18][0], pkv[18][1], pkv[19][0], pkv[19][1],
               pkv[20][0], pkv[20][1], pkv[21][0], pkv[21][1], pkv[22][0], pkv[22][1], pkv[23][0], pkv[23][1], pkv[24][0], pkv[24][1],
               pkv[25][0], pkv[25][1], pkv[26][0], pkv[26][1], pkv[27][0], pkv[27][1], pkv[28][0], pkv[28][1], pkv[29][0], pkv[29][1],
               pkv[30][0], pkv[30][1], pkv[31][0], pkv[31][1] ]

with torch.no_grad():

    dummy_input = (torch.tensor([[1, 2, 3]], dtype=torch.int64),
                  torch.tensor([[1, 1, 1, 1, 1, 1]], dtype=torch.int64),
                  torch.tensor([[3, 4, 5]], dtype=torch.int64))
    for i in range(64):
        dummy_input += (torch.randn(1, 8, 3, 128),)
    input_names = [ "input_ids", "attention_mask", "position_ids",
                "pkv0", "pkv1", "pkv2", "pkv3", "pkv4", "pkv5", "pkv6", "pkv7", "pkv8", "pkv9", "pkv10",
                "pkv11", "pkv12", "pkv13", "pkv14", "pkv15", "pkv16", "pkv17", "pkv18", "pkv19", "pkv20",
                "pkv21", "pkv22", "pkv23", "pkv24", "pkv25", "pkv26", "pkv27", "pkv28", "pkv29", "pkv30",
                "pkv31", "pkv32", "pkv33", "pkv34", "pkv35", "pkv36", "pkv37", "pkv38", "pkv39", "pkv40",
                "pkv41", "pkv42", "pkv43", "pkv44", "pkv45", "pkv46", "pkv47", "pkv48", "pkv49", "pkv50",
                "pkv51", "pkv52", "pkv53", "pkv54", "pkv55", "pkv56", "pkv57", "pkv58", "pkv59", "pkv60",
                "pkv61", "pkv62", "pkv63" ]

    output_names = [ "logits",
                    "opkv0", "opkv1", "opkv2", "opkv3", "opkv4", "opkv5", "opkv6", "opkv7", "opkv8", "opkv9", "opkv10",
                    "opkv11", "opkv12", "opkv13", "opkv14", "opkv15", "opkv16", "opkv17", "opkv18", "opkv19", "opkv20",
                    "opkv21", "opkv22", "opkv23", "opkv24", "opkv25", "opkv26", "opkv27", "opkv28", "opkv29", "opkv30",
                    "opkv31", "opkv32", "opkv33", "opkv34", "opkv35", "opkv36", "opkv37", "opkv38", "opkv39", "opkv40",
                    "opkv41", "opkv42", "opkv43", "opkv44", "opkv45", "opkv46", "opkv47", "opkv48", "opkv49", "opkv50",
                    "opkv51", "opkv52", "opkv53", "opkv54", "opkv55", "opkv56", "opkv57", "opkv58", "opkv59", "opkv60",
                    "opkv61", "opkv62", "opkv63" ]

    torch.onnx.export(LlamaModel(model), dummy_input, "/Users/Vito/Desktop/Mistral7BInst/model.onnx", verbose=False,
        input_names=input_names, output_names=output_names,
        opset_version=14, do_constant_folding=True, export_params=True,
        dynamic_axes={
                'input_ids': {1: 'dim0'},
                'attention_mask': {1: 'dim1'},
                'position_ids': {1: 'dim2'},
                'pkv0': {2: 'dim3'}, 'pkv1': {2: 'dim3'}, 'pkv2': {2: 'dim3'}, 'pkv3': {2: 'dim3'}, 'pkv4': {2: 'dim3'},
                'pkv5': {2: 'dim3'}, 'pkv6': {2: 'dim3'}, 'pkv7': {2: 'dim3'}, 'pkv8': {2: 'dim3'}, 'pkv9': {2: 'dim3'},
                'pkv10': {2: 'dim3'}, 'pkv11': {2: 'dim3'}, 'pkv12': {2: 'dim3'}, 'pkv13': {2: 'dim3'}, 'pkv14': {2: 'dim3'},
                'pkv15': {2: 'dim3'}, 'pkv16': {2: 'dim3'}, 'pkv17': {2: 'dim3'}, 'pkv18': {2: 'dim3'}, 'pkv19': {2: 'dim3'},
                'pkv20': {2: 'dim3'}, 'pkv21': {2: 'dim3'}, 'pkv22': {2: 'dim3'}, 'pkv23': {2: 'dim3'}, 'pkv24': {2: 'dim3'},
                'pkv25': {2: 'dim3'}, 'pkv26': {2: 'dim3'}, 'pkv27': {2: 'dim3'}, 'pkv28': {2: 'dim3'}, 'pkv29': {2: 'dim3'},
                'pkv30': {2: 'dim3'}, 'pkv31': {2: 'dim3'}, 'pkv32': {2: 'dim3'}, 'pkv33': {2: 'dim3'}, 'pkv34': {2: 'dim3'},
                'pkv35': {2: 'dim3'}, 'pkv36': {2: 'dim3'}, 'pkv37': {2: 'dim3'}, 'pkv38': {2: 'dim3'}, 'pkv39': {2: 'dim3'},
                'pkv40': {2: 'dim3'}, 'pkv41': {2: 'dim3'}, 'pkv42': {2: 'dim3'}, 'pkv43': {2: 'dim3'}, 'pkv44': {2: 'dim3'},
                'pkv45': {2: 'dim3'}, 'pkv46': {2: 'dim3'}, 'pkv47': {2: 'dim3'}, 'pkv48': {2: 'dim3'}, 'pkv49': {2: 'dim3'},
                'pkv50': {2: 'dim3'}, 'pkv51': {2: 'dim3'}, 'pkv52': {2: 'dim3'}, 'pkv53': {2: 'dim3'}, 'pkv54': {2: 'dim3'},
                'pkv55': {2: 'dim3'}, 'pkv56': {2: 'dim3'}, 'pkv57': {2: 'dim3'}, 'pkv58': {2: 'dim3'}, 'pkv59': {2: 'dim3'},
                'pkv60': {2: 'dim3'}, 'pkv61': {2: 'dim3'}, 'pkv62': {2: 'dim3'}, 'pkv63': {2: 'dim3'},
        })
```
