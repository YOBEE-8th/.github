# 음성 제어 & 질문

## 1. 음성 제어

### 1.1 결과물 (GIF)

### 마지막 페이지로 이동

![KakaoTalk_20230406_132111757.gif](%E1%84%8B%E1%85%B3%E1%86%B7%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%8C%E1%85%A6%E1%84%8B%E1%85%A5%20&%20%E1%84%8C%E1%85%B5%E1%86%AF%E1%84%86%E1%85%AE%E1%86%AB%20a74ed746338b49d3895051cc30f96561/KakaoTalk_20230406_132111757.gif)

### 이전 페이지로 이동

![KakaoTalk_20230406_132148231.gif](%E1%84%8B%E1%85%B3%E1%86%B7%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%8C%E1%85%A6%E1%84%8B%E1%85%A5%20&%20%E1%84%8C%E1%85%B5%E1%86%AF%E1%84%86%E1%85%AE%E1%86%AB%20a74ed746338b49d3895051cc30f96561/KakaoTalk_20230406_132148231.gif)

---

### 타이머 설정

![KakaoTalk_20230406_135627198.gif](%E1%84%8B%E1%85%B3%E1%86%B7%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%8C%E1%85%A6%E1%84%8B%E1%85%A5%20&%20%E1%84%8C%E1%85%B5%E1%86%AF%E1%84%86%E1%85%AE%E1%86%AB%20a74ed746338b49d3895051cc30f96561/KakaoTalk_20230406_135627198.gif)

---

### 볼륨 조절

![KakaoTalk_20230406_135747136.gif](%E1%84%8B%E1%85%B3%E1%86%B7%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%8C%E1%85%A6%E1%84%8B%E1%85%A5%20&%20%E1%84%8C%E1%85%B5%E1%86%AF%E1%84%86%E1%85%AE%E1%86%AB%20a74ed746338b49d3895051cc30f96561/KakaoTalk_20230406_135747136.gif)

### 1.2 코드

```kotlin
@AndroidEntryPoint
class CookProgressFragment :
    BaseFragment<FragmentCookProgressBinding>(R.layout.fragment_cook_progress) {
    private lateinit var cookProgressSlideAdapter: CookProgressSlideAdapter
    private val cookProgressViewModel by navGraphViewModels<CookProgressViewModel>(R.id.cookProgressGraph) {
        defaultViewModelProviderFactory
    }

    private lateinit var speechRecognizer: SpeechRecognizer
    private lateinit var recognitionListener: RecognitionListener
    private lateinit var speechIntent: Intent
 
    override fun initViewModels() {
        initGetAnswerViewModel()
    }

    override fun onStart() {
        super.onStart()
        initSpeech()
    }

    override fun onStop() {
        super.onStop()
        speechRecognizer.destroy()
        if (tts.isSpeaking) tts.stop()
    }

    // 음성인식 초기화
    private fun initSpeech() {
        SpeechUtils.getPermission(object : PermissionListener {
            override fun onPermissionGranted() {
                speechIntent = Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH)
                speechIntent.putExtra(
                    RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM
                )
                speechIntent.putExtra(
                    RecognizerIntent.EXTRA_CALLING_PACKAGE, requireContext().packageName
                )
                speechIntent.putExtra(RecognizerIntent.EXTRA_LANGUAGE, "ko-KR")

                speechRecognizer = SpeechRecognizer.createSpeechRecognizer(requireContext())
                speechRecognizer.setRecognitionListener(recognitionListener)
                speechRecognizer.startListening(speechIntent)
                cookProgressViewModel.setSttStatusValue(true)
            }

            override fun onPermissionDenied(deniedPermissions: MutableList<String>?) {
                requireContext().showToast(resources.getString(R.string.content_permission_denied))
            }
        })
    }

    // 조리과정 Q&A 통신
    private fun getAnswer(message: String) {
        cookProgressViewModel.getAnswer(message)
    }

    private fun initGetAnswerViewModel() {
        cookProgressViewModel.getAnswerLiveData.observe(this) { response ->
            when (response) {
                is ViewState.Loading -> {
                    Log.d(TAG, "음성 인식한 결과값 통신중")
                }
                is ViewState.Success -> {
                    when (response.value!!.type) {
                        "0" -> { // 화면 이동
                            when (response.value!!.text) {
                                "0" -> { // 처음 페이지로 이동
                                    if (binding.vpCookProgress.currentItem == 0) requireContext().showToast(
                                        resources.getString(R.string.content_answer_first_page)
                                    )
                                    else binding.vpCookProgress.currentItem = 0
                                }
                                "1" -> { // 이전 페이지로 이동
                                    if (binding.vpCookProgress.currentItem == 0) requireContext().showToast(
                                        resources.getString(R.string.content_answer_first_page)
                                    )
                                    else binding.vpCookProgress.currentItem -= 1
                                }
                                "2" -> { // 다음 페이지로 이동
                                    if (binding.vpCookProgress.currentItem == cookProgressSlideAdapter.itemCount - 2) requireContext().showToast(
                                        resources.getString(R.string.content_answer_last_page)
                                    )
                                    else binding.vpCookProgress.currentItem += 1
                                }
                                "3" -> { // 마지막 페이지로 이동
                                    if (binding.vpCookProgress.currentItem == cookProgressSlideAdapter.itemCount - 2) requireContext().showToast(
                                        resources.getString(R.string.content_answer_last_page)
                                    ) else binding.vpCookProgress.currentItem =
                                        cookProgressSlideAdapter.itemCount - 2
                                }
                                "4" -> {
                                    tts.speak(
                                        cookProgressViewModel.cookProgressList.value!!.value!![binding.vpCookProgress.currentItem].description,
                                        TextToSpeech.QUEUE_FLUSH,
                                        null,
                                        null
                                    )
                                }
                            }
                            speechRecognizerStart()
                        }
                        "1" -> { // 질문 답변
                            val answerDialog = AnswerDialog(requireContext(), response.value!!.text)
                            answerDialog.show()
                            tts.speak(response.value!!.text, TextToSpeech.QUEUE_FLUSH, null, null)
                            CoroutineScope(Dispatchers.Main).launch {
                                while (tts.isSpeaking) {
                                    delay(100)
                                }
                                delay(2000)
                                answerDialog.dismiss()
                                speechRecognizer.startListening(speechIntent)
                            }

                            answerDialog.setOnDismissListener {
                                if (tts.isSpeaking) {
                                    tts.stop()
                                }
                            }
                        }
                        "2" -> {
                            NotificationUtils.getPermission(object : PermissionListener {
                                override fun onPermissionGranted() {
                                    cookProgressViewModel.apply {
                                        if(cookProgressList.value?.value?.get(currentPosition)?.timer == 0){
                                            return
                                        }
                                        requireContext().showToast("타이머가 설정되었습니다.")
                                        val alarmFunctions = AlarmFunctions(requireContext())
                                        alarmFunctions.callAlarm(cookProgressList.value?.value?.get(currentPosition)?.timer!!,
                                            System.currentTimeMillis().toInt(),
                                            "요비 타이머")
                                        cookProgressViewModel.setTimerTtsText("타이머가 설정되었습니다.")
                                        Log.d(TAG, "onPermissionGranted: ")
                                    }

                                }

                                override fun onPermissionDenied(deniedPermissions: MutableList<String>?) {
                                    requireContext().showToast("권한을 허용해주세요.")
                                    Log.d(TAG, "onPermissionDenied: ")
                                }
                            })
                            speechRecognizerStart()
                        }
                        "3" -> { // 음성 볼륨 조절
                            val mAudioManager: AudioManager = mainActivity.getSystemService(AUDIO_SERVICE) as AudioManager
                            response.value!!.text.let { text ->
                                val currentVolume = mAudioManager.getStreamVolume(AudioManager.STREAM_MUSIC)
                                when(text){
                                    "up" -> {
                                        mAudioManager.setStreamVolume(AudioManager.STREAM_MUSIC,currentVolume+4,AudioManager.FLAG_SHOW_UI)
                                    }
                                    "down" -> {
                                        mAudioManager.setStreamVolume(AudioManager.STREAM_MUSIC,currentVolume-4,AudioManager.FLAG_SHOW_UI)
                                    }
                                }
                            }
                            speechRecognizerStart()
                        }
                    }
                    Log.d(TAG, "음성 인식 통신 성공 : ${response.value!!.type} ${response.value!!.text}")
                }
                is ViewState.Error -> {
                    requireContext().showToast(response.message!!)
                    Log.d(TAG, "음성 인식 통신 실패")
                    speechRecognizerStart()
                }
            }
        }
    }

    // 음성 인식 리스너 등록
    private fun setListener() {
        recognitionListener = object : RecognitionListener {

            override fun onReadyForSpeech(params: Bundle?) {
                cookProgressViewModel.setSttStatusValue(true)
            }

            override fun onBeginningOfSpeech() {

            }

            override fun onRmsChanged(rmsdB: Float) {

            }

            override fun onBufferReceived(buffer: ByteArray?) {

            }

            override fun onEndOfSpeech() {
                cookProgressViewModel.setSttStatusValue(false)
            }

            override fun onError(error: Int) {
                var message = ""
                when (error) {
                    SpeechRecognizer.ERROR_AUDIO -> message =
                        resources.getString(R.string.content_error_audio)
                    SpeechRecognizer.ERROR_CLIENT -> Log.d(
                        TAG, "ERROR_CLIENT: ${resources.getString(R.string.content_error_client)}"
                    )
                    SpeechRecognizer.ERROR_INSUFFICIENT_PERMISSIONS -> message =
                        resources.getString(R.string.content_error_insufficient_permissions)
                    SpeechRecognizer.ERROR_NETWORK -> message =
                        resources.getString(R.string.content_error_network)
                    SpeechRecognizer.ERROR_NETWORK_TIMEOUT -> message =
                        resources.getString(R.string.content_error_network_time_out)
                    SpeechRecognizer.ERROR_NO_MATCH -> Log.d(
                        TAG,
                        "ERROR_NO_MATCH: ${resources.getString(R.string.content_error_no_match)}"
                    )
                    SpeechRecognizer.ERROR_SERVER -> Log.d(
                        TAG, "ERROR_SERVER: ${resources.getString(R.string.content_error_server)}"
                    )
                    SpeechRecognizer.ERROR_SPEECH_TIMEOUT -> message =
                        resources.getString(R.string.content_error_speech_time_out)
                    else -> message = resources.getString(R.string.content_error_other)
                }
                if (message.isNotEmpty()) requireContext().showToast(message)
                speechRecognizerStart()
            }

            override fun onResults(results: Bundle?) {
                var matches: ArrayList<String> =
                    results?.getStringArrayList(SpeechRecognizer.RESULTS_RECOGNITION) as ArrayList<String>

                if (cookProgressViewModel.isKeywordInActivate) { // 키워드 없이 말하는게 활성화된 상태
                    getAnswer(matches[0])
                    commandDialog.dismiss()

                } else {
                    if (!tts.isSpeaking) {
                        if (matches[0].contains(cookProgressViewModel.voiceName)) { // 특정 단어 인식하면 요청
                            val question = matches[0].split(" ")
                            val sb = StringBuilder()

                            out@ for (i in question.size-1 downTo 0) {
                                if (question[i].contains(cookProgressViewModel.voiceName)) {
                                    for (j in i until question.size) sb.append("${question[j]} ")
                                    break@out
                                }
                            }

                            getAnswer(sb.toString())
                        } else {
                            speechRecognizerStart()
                        }
                    } else {
                        speechRecognizerStart()
                    }
                }

                cookProgressViewModel.setKeywordExist(false)
                Log.d(TAG, "onResults: ${matches[0]}")
            }

            override fun onPartialResults(partialResults: Bundle?) {

            }

            override fun onEvent(eventType: Int, params: Bundle?) {

            }

        }
    }

    // 키워드 명령어 비활성화
    fun inActivateKeyWord() {
        if (tts.isSpeaking) {
            tts.stop()
        }

        commandDialog =
            AnswerDialog(requireContext(), resources.getString(R.string.content_question_help))
        commandDialog.show()
        cookProgressViewModel.setKeywordExist(true)
    }

    fun speechRecognizerStart() = CoroutineScope(Dispatchers.Main).launch {
        delay(1000)
        speechRecognizer.startListening(speechIntent)
        cookProgressViewModel.setSttStatusValue(true)
    }
}
```

### 1.3 코드 설명

- **STT 엔진은 구글 STT 엔진을 사용하였으며, `SpeechRecognizer`을 통해 STT로 변형된 텍스트를 바탕으로 음성제어 기능을 구현하였습니다.**
- **조리 시작 페이지에서 선택한 음성 비서 이름을 키워드로 인식하여서 해당 요리 비서 이름이 들어가면 가장 최근에 말한 키워드 기준으로 키워드를 포함해 뒤에 있는 내용을 서버에 요청합니다.**
- **서버에서 통신하는 동안에는 `SpeechRecognizer`을 실행시키지 않고, 서버 통신이 끝나거나 음성인식이 제대로 되지 않았을 때 다시 음성 인식을 자동으로 시작하게 구현하였습니다.**
- **`onStop()` 과정에서는 음성인식이 되면 안되기 때문에 destroy하고 `onStart()`에서 다시 `SpeechRecognizer`을 생성하여 시작합니다.**
- **서버에서 통신한 결과값을 `when문`을 통해 해당 결과값에 맞는 기능을 실행하도록 구현하였습니다.**
-