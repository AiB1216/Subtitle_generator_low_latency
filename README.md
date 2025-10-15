# Subtitle_generator_low_latency
Speech to Text (STT) via concurrent execution of 2 threads

Project nature:

- .ipynb based coding
- Dependency (Primary):

  - faster_whisper
  - soundcard
  - scipy.io.wavfile
  - threading

Key code:
```python
def looped_transcribe(model, n_chunks, duration, test_queue = False, print_true = False, save_wav=False):
    global init_time
    init_time, file_queue = time.time(), Queue(); print("Queue created", end = "\r")
    loopback_mic = sc.get_microphone(sc.default_speaker().name, include_loopback=True)
    with open("Record_log.txt", "w") as f: 
        pass # log clear
    
    if not test_queue: 
        record_thread_1 = threading.Thread(target=record, args=(loopback_mic, n_chunks, duration, save_wav, file_queue, 1))
        record_thread_2 = threading.Thread(target=record, args=(loopback_mic, n_chunks, duration, save_wav, file_queue, 2))
    else:
        clear_output(); print("file_queue:", end = " ")
        for filename in [i for i in os.listdir(".") if i.endswith(".wav")]:
            file_queue.put((filename, 0)); print(filename, end=", ")
        file_queue.put(None); print("\n")
    stt_thread = threading.Thread(target=STT, args=(model, duration, print_true, file_queue))

    if not test_queue: record_thread_1.start(); time.sleep(1); record_thread_2.start()
    stt_thread.start()

    if not test_queue: record_thread_1.join(); time.sleep(1); record_thread_2.join()
    stt_thread.join()
```
