# Tensorflow Object Detection 

- 텐서플로우 모델이 복사되어 있고, 설정 되어있는 상태에서 시작한다.

### 데이터 라벨링

1. LabelImg를 설치한다.

```
> pip install labelImg
```

2. cmd 창에 labelImg 입력 후 실행할 수 있다.

3. 실행 후, Open Dir과 Change Save Dir을 데이터 사진들이 들어있는 폴더로 지정하고, 라벨링을 시작하면 된다.

   ※ 이미지 이름에 한글이 있다거나, 너무 길다거나, '.' 같은 것들이 들어있으면 나중에 레코드를 만들 때 문제가 생기는 것 같다. 미리 이름을 바꿔주자.

4. Rect box를 그린 후, 객체 이름을 정해준다. 이 때, 띄어쓰기랑 오타에 주의하자!

   | 단축키 | 내용            |
   | ------ | --------------- |
   | ctrl+s | 저장            |
   | ctrl+d | Rect Box 복제   |
   | a, d   | 이전 / 다음     |
   | w      | 새로운 Rect Box |

5. object_detection 폴더에 images 라는 새로운 폴더를 만들고, 그 안에 train, test 라는 폴더를 각각 만들어준다.

   ```
   object_detection
   └─images
      ├─ train
      └─ test
   ```

   

6. 데이터 라벨링이 끝난 이미지들을 적절한 비율로 나누어서 train, test 폴더에 각각 담는다. (train : test = 9 : 1 비율로 나누는 것이 정석이라고 한다.)

### .xml -> .csv

- 참고 : https://github.com/datitran/raccoon_dataset

1. xml_to_csv.py 파일을 object_detection 폴더에 넣는다. (url 참고)

2. main 함수를 살짝 수정해준다.

   ```python
   def main():
       for directory in ['train', 'test']:
           image_path = os.path.join(os.getcwd(), 'images/{}'.format(directory))
           xml_df = xml_to_csv(image_path)
           xml_df.to_csv('data/{}_labels.csv'.format(directory), index=None)
           print('Successfully converted xml to csv.')
   ```

3. xml_to_csv.py 파일을 실행하면 object_detection/data 폴더에 test_labels.csv, train_labels.csv가 생성된다.

### .csv -> .record

1. generate_tfrecord.py 파일을 object_detection 폴더에 넣는다. (url 참고)

2. class_text_to_int 의 내용을 수정한다. 자신의 데이터셋에 들어있는 라벨들을 모두 넣어준다.

   ```python
   def class_text_to_int(row_label):
       if row_label == 'dog':
       	return 1
       elif row_label == 'cat':
       	return 2
       else:
       	None
   ```

3. cmd에서 아래 명령어 2개를 실행한다.

   ```
   > python generate_tfrecord.py --csv_input=data/train_labels.csv --output_path=data/train.record --image_dir=images/train
   > python generate_tfrecord.py --csv_input=data/test_labels.csv --output_path=data/test.record --image_dir=images/test
   ```

4. 실행하면 object_detection/data 폴더에 test.record, train.record 파일이 생성된다.

### 학습시키기

1. pre-trained model을 준비해놓는다. 다운 받은 후, object_detection 폴더에 압축을 풀어둔다.

   (http://download.tensorflow.org/models/object_detection/ssd_mobilenet_v1_coco_11_06_2017.tar.gz)

2. object_detection 폴더에 training 폴더를 만든다.
3. object_detection/samples/configs 내에 있는 ssd_mobilenet_v1_pets.config 파일을 training 폴더로 복사한다.

4. ssd_mobilenet_v1_pets.config 파일을 수정한다.(num_classes, batch_size, fine_tune_checkpoint, train_input_reader, eval_input_reader )

   - ```
     model{
     	ssd{
     		num_classes:5 //감지해야 할 라벨의 개수
     		box_coder{
     			faster_rcnn_box_coder{ ...}
     		}
     	}
     }
     ```

   - ```
     train_config:{
     	batch_size:16 //더 높게 하면 정확도는 높아지지만 시간이 오래걸린다.
     	optimizer{
     		...
     	}
     	...
     }
     ```

   - ```
     fine_tune_checkpoint:
     "ssd_mobilenet_v1_coco_11_06_2017/model.ckpt" //다운받은 모델로 변경해준다.
     ```

   - ```
     train_input_reader:{
     	tf_record_input_reader{
     		input_path:"data/train.record" // 변경
     	}
     	label_map_path:"data/object-detection.pbtxt" // 변경
     }
     ...
     eval_input_reader:{
     	tf_record_input_reader{
     		input_path:"data/test.record" // 변경
     	}
     	label_map_path:"data/object-detection.pbtxt" // 변경
     	shuffle:false
     	num_readers: 1
     }
     ```

5. object_detection/data 폴더에 object-detection.pbtxt 파일을 만든다. 라벨 id와 이름은 정했던 대로 작성한다.

   ```
   item{
   	id:1
   	name:'dog'
   }
   item{
   	id:2
   	name:'cat'
   }
   ```

6. 이제 학습을 진행한다. object_detectionn/legacy/train.py 파일을 이용할 것이다. cmd에서 object_detection 폴더 경로에 있는 채로 아래 명령어를 실행한다.

   ```
   python legacy/train.py --logtostderr --train_dir=training/ --pipeline_config_path=training/ssd_mobilenet_v1_pets.config
   ```

7. cmd에 loss값이 계속 나오는데, 원하는 정도로 돌리다가 종료하면 된다. 0에 가까워져 있어야 하긴 한다..

8. 정확도 있는 결과를 얻으려면 10000번은 해야 하는 것 같다..

9. tensorboard로 그래프를 확인할 수 있다. cmd의 object_detection 폴더에서 아래 명령어를 실행하면 tensorboard를 열 수 있다.

   ```
   tensorboard --logdir=./training
   ```

### freeze_inference_graph.pb가 필요하다면?

1. freeze_inferennce_graph.pb 파일은 학습이 완료된 그래프를 Export 해놓은 파일이다. 이 파일을 통해 예측을 할 수 있다.

2. object_detection 폴더에서 아래 명령어를 실행하면, object_detection/new 파일에 결과가 생성된다.

   ```
   python export_inference_graph.py --input_type=image_tensor --pipeline_config_path=training/ssd_mobilenet_v1_pets.config --trained_checkpoint_prefix=training/model.ckpt-838 --output_directory=new
   ```

   

