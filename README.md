# KE_VIST
The code and output of our AAAI paper "Knowledge-Enriched Visual Storytelling" ([arxiv](https://arxiv.org/abs/1912.01496))


## generated_stories
Unlike the format in VIST dataset, here we put all stories in a column. E.g.

```
{
        "original_text": "The local parish holds a craft show each year.",
        "album_id": "44277",
        "photo_flickr_id": "1741642",
        "story_id": "45530",
        "text_mapped_with_nouns_and_frame": [
            "parish_NOUN",
            "Containing_Frame",
            "craft_NOUN",
            "show_NOUN",
            "year_NOUN"
        ],
        "predicted_term_seq": [
            "Arriving_Frame",
            "town_NOUN"
        ],
        "predicted_story": "i was out of town . there were so many books to find . the table looked very old . we played games . it came from a man ."
```

`predicted_term_seq` is either the terms predicted from our image2term module(in this example) or . `predicted_story` is the whole story predicted by our term2story model, and `text_mapped_with_nouns_and_frame` is the result open-seasame extracted from original sentence.


## Stage 1: Word distillation from input prompts
### Environment
```
pytorch==1.3
python==3.6
Caffe
```
1. in ```src/stage1/image_preprocess``` resize images into 224*224
```
bash resize_all_image.sh CPU_NUM IMAGE_DIR OUTPUT_DIR
```
2.in ```src/stage1/image_object_detection``` perform object detection & object feature extraction
With Caffe environment
```
# Object Detection
python3.6 ./tools/generate_tsv.py --gpu 0,1 \
--cfg experiments/cfgs/faster_rcnn_end2end_resnet.yml\
--def ./models/vg/ResNet-101/faster_rcnn_end2end_final/test.prototxt\
--out vist_with_classes_attr_png.csv\
--net data/faster_rcnn_models/resnet101_faster_rcnn_final.caffemodel\
--split vist

# Split file into npzs 
python3.6 make_bu_data.py
```
3.in ```src/stage1/image2term``` generate terms from image feature 
```
# Build Vocabulary
python3.6 build_term_vocab.py

#Training
# bash run.sh GPU_ID Decode_mode[transformer, rnn] Decoder_type[term, text] [-self_att] [-global_att]
bash run_train.sh 0 term rnn -self_att
```

## Stage 2: Word enrichment using knowledge graphs
### Environment
```
pytorch==1.3
python==3.6
```
### Usage
1. in ```src/stage2/data``` execute ```./download_big_data.sh``` to download those files > 100MB
2. to train the path scoring language model
```
python3.6 main.py
```
trained model will be saved as ```./model.pt```

3 to link paths using Visual Genome Knowledge Graph i.e. **Visual_Genome** in the paper
```
python score_relation.vg.py
```
result story_id-to-path file will be saved as ```./vist_scored_terms_6_path.json``` (The path in this file is different from that in the generated stories since the model has been trained again)

For the openIE part, I mis-delete the knowledge graph result of OpenIE. I will try to solve it in the future.

## Stage 3: Story generation
