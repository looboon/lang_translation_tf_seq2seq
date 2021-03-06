# Language Translation using Tensorflow's new seq2seq Library
Given that Tensorflow 1.2 has recently released a new library for sequence to sequence models and has made some of the old codes obsolete, this is an attempt to redo the Udacity Deep Learning Nanodegree Assigment 4 - Language Translation from English to French using the new seq2seq library. 

Some changes were: 
- old APIs (simple_decoder_fn_train and dynamic_rnn_decoder) were replaced
- new APIs (TrainingHelper/GreedyEmbeddingHelper, BasicDecoder and dynamic_decode) to create the training and inference decoders
- additional placeholders needed to support the new APIs (source_sequence_length and target_sequence_length)

I also added some additional features to make this model seem more like the model in the paper [Neural Machine Translation by Jointly Learning to Align and Translate](https://arxiv.org/abs/1409.0473) by introducing:
- Bidirectional LSTMs using tf.nn.bidirectional_dynamic_rnn. Using bidirectional LSTMs make sense as in the context of translation we do not only look backward but also forward as the use of a word at the back may influence how its used in the beginning (think grammar and vocab rules).
```
    enc_output, enc_state = tf.nn.bidirectional_dynamic_rnn(
                        cell_fw=enc_cell,
                        cell_bw=enc_cell,
                        sequence_length=source_sequence_length,
                        inputs=enc_embed_input,
                        dtype=tf.float32) 
```
- Bahdanau attention (additive) using a few functions as show below. Unlike the original seq2seq model by [Sutskever et al.](https://papers.nips.cc/paper/5346-sequence-to-sequence-learning-with-neural-networks.pdf) where the context vector of the encoder-decoder is fixed, the attention mechanism makes it such that every position in the output sequence has its own context vector. This allows the decoder to have a peek at every position in the encoder and builds up even longer term dependencies than with purely LSTMs.
```    
    attention_mechanism = tf.contrib.seq2seq.BahdanauAttention(num_units=rnn_size,
                    memory=encoder_output, memory_sequence_length=source_sequence_length)
    
    attn_cell = tf.contrib.seq2seq.AttentionWrapper(dec_cell, attention_mechanism, 
                                                   attention_layer_size=rnn_size)
    
    out_cell = tf.contrib.rnn.OutputProjectionWrapper(attn_cell, vocab_size, reuse=False) 
``` 
To do (if I have time):
- add beam search decoder instead of the current greedy decoding (where the output word is just chosen based on max probability). In this way, we consider top K contenders at each position instead of the top contender only. This makes more sense as the top word for position 1 may not be the top word for the entire sequence generated. It might be the second or third from top instead. See this [link](https://www.quora.com/Why-is-beam-search-required-in-sequence-to-sequence-transduction-using-recurrent-neural-networks) for more details.
- train on bigger corpus (need GPU hours though)


