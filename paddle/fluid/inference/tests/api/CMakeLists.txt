set(INFERENCE_EXTRA_DEPS paddle_inference_api paddle_fluid_api ir_pass_manager analysis_predictor benchmark)

if(WITH_GPU AND TENSORRT_FOUND)
    set(INFERENCE_EXTRA_DEPS ${INFERENCE_EXTRA_DEPS} analysis ${analysis_deps} ir_pass_manager analysis_predictor)
endif()

function(download_model install_dir model_name)
    if (NOT EXISTS ${install_dir})
        inference_download_and_uncompress(${install_dir} ${INFERENCE_URL} ${model_name})
    endif()
endfunction()

function(download_model_and_data install_dir model_name data_name)
    if (NOT EXISTS ${install_dir})
        inference_download_and_uncompress(${install_dir} ${INFERENCE_URL} ${model_name})
        inference_download_and_uncompress(${install_dir} ${INFERENCE_URL} ${data_name})
    endif()
endfunction()

function(inference_analysis_api_test target install_dir filename)
    inference_analysis_test(${target} SRCS ${filename}
        EXTRA_DEPS ${INFERENCE_EXTRA_DEPS} benchmark
        ARGS --infer_model=${install_dir}/model --infer_data=${install_dir}/data.txt)
endfunction()

function(inference_analysis_api_int8_test target model_dir data_dir filename)
    inference_analysis_test(${target} SRCS ${filename}
        EXTRA_DEPS ${INFERENCE_EXTRA_DEPS} benchmark
        ARGS --infer_model=${model_dir}/model
             --infer_data=${data_dir}/data.bin
             --warmup_batch_size=100
             --batch_size=50
             --paddle_num_threads=${CPU_NUM_THREADS_ON_CI}
	     --iterations=2)
endfunction()

function(inference_analysis_api_test_with_fake_data target install_dir filename model_name)
    download_model(${install_dir} ${model_name})
    inference_analysis_test(${target} SRCS ${filename}
        EXTRA_DEPS ${INFERENCE_EXTRA_DEPS}
        ARGS --infer_model=${install_dir}/model)
endfunction()

function(inference_analysis_api_test_with_refer_result target install_dir filename)
    inference_analysis_test(${target} SRCS ${filename}
        EXTRA_DEPS ${INFERENCE_EXTRA_DEPS}
        ARGS --infer_model=${install_dir}/model --infer_data=${install_dir}/data.txt
             --refer_result=${install_dir}/result.txt)
endfunction()

if(NOT APPLE AND WITH_MKLML)
    # RNN1
    set(RNN1_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/rnn1")
    download_model_and_data(${RNN1_INSTALL_DIR} "rnn1%2Fmodel.tar.gz" "rnn1%2Fdata.txt.tar.gz")
    inference_analysis_api_test(test_analyzer_rnn1 ${RNN1_INSTALL_DIR} analyzer_rnn1_tester.cc)
    
    # seq_pool1
    set(SEQ_POOL1_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/seq_pool")
    download_model_and_data(${SEQ_POOL1_INSTALL_DIR} "seq_pool1_model_.tar.gz" "seq_pool1_data.txt.tar.gz")
    inference_analysis_api_test(test_analyzer_seq_pool1 ${SEQ_POOL1_INSTALL_DIR} analyzer_seq_pool1_tester.cc)
else()
    # TODO: fix this test on MACOS and OPENBLAS, the reason is that
    # fusion_seqexpand_concat_fc_op is not supported on MACOS and OPENBLAS
    message(WARNING "These tests has been disabled in OSX or WITH_MKL=OFF before being fixed: \n test_analyzer_rnn1")
    message(WARNING "These tests has been disabled in OSX or WITH_MKL=OFF before being fixed: \n test_analyzer_seq_pool1")
endif()


# RNN2
set(RNN2_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/rnn2")
download_model_and_data(${RNN2_INSTALL_DIR} "rnn2_model.tar.gz" "rnn2_data.txt.tar.gz")
inference_analysis_api_test(test_analyzer_rnn2 ${RNN2_INSTALL_DIR} analyzer_rnn2_tester.cc)

# TODO(luotao, Superjom) Disable DAM test, temporarily fix
# https://github.com/PaddlePaddle/Paddle/issues/15032#issuecomment-455990914.
# After inference framework refactor, will reopen it.
# normal DAM
set(DAM_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/dam")
download_model_and_data(${DAM_INSTALL_DIR} "DAM_model.tar.gz" "DAM_data.txt.tar.gz")
#inference_analysis_api_test(test_analyzer_dam ${DAM_INSTALL_DIR} analyzer_dam_tester.cc EXTRA_DEPS legacy_allocator)

# small DAM
set(DAM_SMALL_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/small_dam")
download_model_and_data(${DAM_SMALL_INSTALL_DIR} "dam_small_model.tar.gz" "dam_small_data.txt.tar.gz")
inference_analysis_test(test_analyzer_small_dam SRCS analyzer_dam_tester.cc
        EXTRA_DEPS ${INFERENCE_EXTRA_DEPS}
        ARGS --infer_model=${DAM_SMALL_INSTALL_DIR}/model --infer_data=${DAM_SMALL_INSTALL_DIR}/data.txt --max_turn_num=1)

# save model
inference_analysis_api_test(test_analyzer_save_model ${DAM_SMALL_INSTALL_DIR} analyzer_save_model_tester.cc)

# chinese_ner
set(CHINESE_NER_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/chinese_ner")
download_model_and_data(${CHINESE_NER_INSTALL_DIR} "chinese_ner_model.tar.gz" "chinese_ner-data.txt.tar.gz")
inference_analysis_api_test(test_analyzer_ner ${CHINESE_NER_INSTALL_DIR} analyzer_ner_tester.cc)

# lac
set(LAC_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/lac")
download_model_and_data(${LAC_INSTALL_DIR} "lac_model.tar.gz" "lac_data.txt.tar.gz")
inference_analysis_api_test(test_analyzer_lac ${LAC_INSTALL_DIR} analyzer_lac_tester.cc)

# MM DNN
set(MM_DNN_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/mm_dnn")
download_model_and_data(${MM_DNN_INSTALL_DIR} "MM_DNN_model.tar.gz" "MM_DNN_data.txt.tar.gz")
inference_analysis_api_test(test_analyzer_mm_dnn ${MM_DNN_INSTALL_DIR} analyzer_mm_dnn_tester.cc)

# Pyramid DNN
set(PYRAMID_DNN_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/pyramid_dnn")
download_model_and_data(${PYRAMID_DNN_INSTALL_DIR} "PyramidDNN_model.tar.gz" "PyramidDNN_data.txt.tar.gz")
inference_analysis_api_test(test_analyzer_pyramid_dnn ${PYRAMID_DNN_INSTALL_DIR} analyzer_pyramid_dnn_tester.cc)

# text_classification
set(TEXT_CLASSIFICATION_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/text_classification")
download_model_and_data(${TEXT_CLASSIFICATION_INSTALL_DIR} "text-classification-Senta.tar.gz" "text_classification_data.txt.tar.gz")
inference_analysis_api_test(test_analyzer_text_classification ${TEXT_CLASSIFICATION_INSTALL_DIR} analyzer_text_classification_tester.cc)

# seq_conv1
set(SEQ_CONV1_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/seq_conv1")
download_model_and_data(${SEQ_CONV1_INSTALL_DIR} "seq_conv1_model.tar.gz" "seq_conv1_data.txt.tar.gz")
inference_analysis_api_test(test_analyzer_seq_conv1 ${SEQ_CONV1_INSTALL_DIR} analyzer_seq_conv1_tester.cc)

# transformer, the dataset only works on batch_size=8 now
set(TRANSFORMER_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/transformer")
download_model_and_data(${TRANSFORMER_INSTALL_DIR} "temp%2Ftransformer_model.tar.gz" "temp%2Ftransformer_data.txt.tar.gz")
inference_analysis_test(test_analyzer_transformer SRCS analyzer_transformer_tester.cc 
  EXTRA_DEPS ${INFERENCE_EXTRA_DEPS}
  ARGS --infer_model=${TRANSFORMER_INSTALL_DIR}/model --infer_data=${TRANSFORMER_INSTALL_DIR}/data.txt --batch_size=8 
       --paddle_num_threads=${CPU_NUM_THREADS_ON_CI})

# ocr
set(OCR_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/ocr")
if (NOT EXISTS ${OCR_INSTALL_DIR})
    inference_download_and_uncompress(${OCR_INSTALL_DIR} "http://paddlemodels.bj.bcebos.com/" "inference-vis-demos%2Focr.tar.gz")
endif()
inference_analysis_api_test_with_refer_result(test_analyzer_ocr ${OCR_INSTALL_DIR} analyzer_vis_tester.cc)

# mobilenet with transpose op
set(MOBILENET_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/mobilenet")
if (NOT EXISTS ${MOBILENET_INSTALL_DIR})
    inference_download_and_uncompress(${MOBILENET_INSTALL_DIR} "http://paddlemodels.bj.bcebos.com/" "inference-vis-demos%2Fmobilenet.tar.gz")
endif()
inference_analysis_api_test_with_refer_result(test_analyzer_mobilenet_transpose ${MOBILENET_INSTALL_DIR} analyzer_vis_tester.cc)

# googlenet
inference_analysis_api_test_with_fake_data(test_analyzer_googlenet
  "${INFERENCE_DEMO_INSTALL_DIR}/googlenet" analyzer_resnet50_tester.cc "googlenet.tar.gz")

# resnet50
inference_analysis_api_test_with_fake_data(test_analyzer_resnet50
  "${INFERENCE_DEMO_INSTALL_DIR}/resnet50" analyzer_resnet50_tester.cc "resnet50_model.tar.gz")

# mobilenet with depthwise_conv op
inference_analysis_api_test_with_fake_data(test_analyzer_mobilenet_depthwise_conv
  "${INFERENCE_DEMO_INSTALL_DIR}/mobilenet_depthwise_conv" analyzer_resnet50_tester.cc "mobilenet_model.tar.gz")

# int8 image classification tests
if(WITH_MKLDNN)
  set(INT8_DATA_DIR "${INFERENCE_DEMO_INSTALL_DIR}/int8v2")
  if (NOT EXISTS ${INT8_DATA_DIR})
    inference_download_and_uncompress(${INT8_DATA_DIR} "${INFERENCE_URL}/int8" "imagenet_val_100_tail.tar.gz")
  endif()

  #resnet50 int8
  set(INT8_RESNET50_MODEL_DIR "${INT8_DATA_DIR}/resnet50")
  if (NOT EXISTS ${INT8_RESNET50_MODEL_DIR})
    inference_download_and_uncompress(${INT8_RESNET50_MODEL_DIR} "${INFERENCE_URL}/int8" "resnet50_int8_model.tar.gz" )
  endif()
  inference_analysis_api_int8_test(test_analyzer_int8_resnet50 ${INT8_RESNET50_MODEL_DIR} ${INT8_DATA_DIR} analyzer_int8_image_classification_tester.cc)

  #mobilenet int8
  set(INT8_MOBILENET_MODEL_DIR "${INT8_DATA_DIR}/mobilenet")
  if (NOT EXISTS ${INT8_MOBILENET_MODEL_DIR})
    inference_download_and_uncompress(${INT8_MOBILENET_MODEL_DIR} "${INFERENCE_URL}/int8" "mobilenetv1_int8_model.tar.gz" )
  endif()
  inference_analysis_api_int8_test(test_analyzer_int8_mobilenet ${INT8_MOBILENET_MODEL_DIR} ${INT8_DATA_DIR} analyzer_int8_image_classification_tester.cc)
 
  #resnet101 int8
  set(INT8_RESNET101_MODEL_DIR "${INT8_DATA_DIR}/resnet101")
  if (NOT EXISTS ${INT8_RESNET101_MODEL_DIR})
    inference_download_and_uncompress(${INT8_RESNET101_MODEL_DIR} "${INFERENCE_URL}/int8" "Res101_int8_model.tar.gz" )
  endif()
  inference_analysis_api_int8_test(test_analyzer_int8_resnet101 ${INT8_RESNET101_MODEL_DIR} ${INT8_DATA_DIR} analyzer_int8_image_classification_tester.cc)
 
  #vgg16 int8
  set(INT8_VGG16_MODEL_DIR "${INT8_DATA_DIR}/vgg16")
  if (NOT EXISTS ${INT8_VGG16_MODEL_DIR})
    inference_download_and_uncompress(${INT8_VGG16_MODEL_DIR} "${INFERENCE_URL}/int8" "VGG16_int8_model.tar.gz" )
  endif()
  inference_analysis_api_int8_test(test_analyzer_int8_vgg16 ${INT8_VGG16_MODEL_DIR} ${INT8_DATA_DIR} analyzer_int8_image_classification_tester.cc)
 
  #vgg19 int8
  set(INT8_VGG19_MODEL_DIR "${INT8_DATA_DIR}/vgg19")
  if (NOT EXISTS ${INT8_VGG19_MODEL_DIR})
    inference_download_and_uncompress(${INT8_VGG19_MODEL_DIR} "${INFERENCE_URL}/int8" "VGG19_int8_model.tar.gz" )
  endif()
  inference_analysis_api_int8_test(test_analyzer_int8_vgg19 ${INT8_VGG19_MODEL_DIR} ${INT8_DATA_DIR} analyzer_int8_image_classification_tester.cc)
endif()

# bert, max_len=20, embedding_dim=128
set(BERT_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/bert_emb128")
download_model_and_data(${BERT_INSTALL_DIR} "bert_emb128_model.tar.gz" "bert_data_len20.txt.tar.gz")
inference_analysis_api_test(test_analyzer_bert ${BERT_INSTALL_DIR} analyzer_bert_tester.cc)

# anakin
if (WITH_ANAKIN AND WITH_MKL) # only needed in CI
    # anakin rnn1
    set(ANAKIN_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/anakin")
    set(ANAKIN_RNN1_INSTALL_DIR "${ANAKIN_INSTALL_DIR}/rnn1")
    inference_download(${ANAKIN_RNN1_INSTALL_DIR} ${INFERENCE_URL} "anakin_test%2Fditu_rnn.anakin2.model.bin")
    inference_download(${ANAKIN_RNN1_INSTALL_DIR} ${INFERENCE_URL} "anakin_test%2Fditu_rnn_data.txt")
    cc_test(test_anakin_rnn1 SRCS anakin_rnn1_tester.cc
            ARGS --model=${ANAKIN_RNN1_INSTALL_DIR}/anakin_test%2Fditu_rnn.anakin2.model.bin
                 --datapath=${ANAKIN_RNN1_INSTALL_DIR}/anakin_test%2Fditu_rnn_data.txt
            DEPS inference_anakin_api_shared)
    # anakin mobilenet
    if(WITH_GPU)
        set(ANAKIN_MOBILENET_INSTALL_DIR "${ANAKIN_INSTALL_DIR}/mobilenet")
        inference_download(${ANAKIN_MOBILENET_INSTALL_DIR} ${INFERENCE_URL} "mobilenet_v2.anakin.bin")
        cc_test(test_anakin_mobilenet SRCS anakin_mobilenet_tester.cc
                ARGS --model=${ANAKIN_MOBILENET_INSTALL_DIR}/mobilenet_v2.anakin.bin
                DEPS inference_anakin_api_shared dynload_cuda)
    endif()
endif()

if(WITH_GPU AND TENSORRT_FOUND)
    set(TRT_MODEL_INSTALL_DIR "${INFERENCE_DEMO_INSTALL_DIR}/trt")
    if (NOT EXISTS ${TRT_MODEL_INSTALL_DIR})
        inference_download_and_uncompress(${TRT_MODEL_INSTALL_DIR} ${INFERENCE_URL}/tensorrt_test "trt_test_models.tar.gz")
    endif()
    inference_analysis_test(test_trt_models SRCS trt_models_tester.cc
            EXTRA_DEPS ${INFERENCE_EXTRA_DEPS}
            ARGS --infer_model=${TRT_MODEL_INSTALL_DIR}/trt_test_models)
endif()
