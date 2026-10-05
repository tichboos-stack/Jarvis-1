package com.example.jarvis

import android.Manifest
import android.app.Activity
import android.content.Intent
import android.content.pm.PackageManager
import android.net.Uri
import android.os.Bundle
import android.speech.RecognitionListener
import android.speech.RecognizerIntent
import android.speech.SpeechRecognizer
import android.speech.tts.TextToSpeech
import android.widget.Button
import android.widget.LinearLayout
import android.widget.TextView
import java.text.SimpleDateFormat
import java.util.*

class MainActivity : Activity(), TextToSpeech.OnInitListener {

    private lateinit var speechRecognizer: SpeechRecognizer
    private lateinit var textToSpeech: TextToSpeech
    private lateinit var statusText: TextView
    private lateinit var listenButton: Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        if (checkSelfPermission(Manifest.permission.RECORD_AUDIO)
            != PackageManager.PERMISSION_GRANTED) {
            requestPermissions(
                arrayOf(Manifest.permission.RECORD_AUDIO),
                100
            )
        }

        textToSpeech = TextToSpeech(this, this)
        speechRecognizer = SpeechRecognizer.createSpeechRecognizer(this)

        statusText = TextView(this).apply {
            text = "JARVIS तैयार है"
            textSize = 22f
            setPadding(30, 60, 30, 30)
        }

        listenButton = Button(this).apply {
            text = "🎤 JARVIS से बोलें"
            setOnClickListener {
                startListening()
            }
        }

        val layout = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            addView(statusText)
            addView(listenButton)
        }

        setContentView(layout)

        speechRecognizer.setRecognitionListener(
            object : RecognitionListener {

                override fun onReadyForSpeech(params: Bundle?) {
                    statusText.text = "🎤 सुन रहा हूँ..."
                }

                override fun onResults(results: Bundle?) {

                    val commands =
                        results?.getStringArrayList(
                            SpeechRecognizer.RESULTS_RECOGNITION
                        )

                    val command = commands?.firstOrNull() ?: ""

                    statusText.text = "आप: $command"

                    runCommand(command.lowercase(Locale.getDefault()))
                }

                override fun onError(error: Int) {
                    statusText.text = "Command समझ नहीं आई"
                    listenButton.isEnabled = true
                }

                override fun onBeginningOfSpeech() {}
                override fun onRmsChanged(rmsdB: Float) {}
                override fun onBufferReceived(buffer: ByteArray?) {}
                override fun onEndOfSpeech() {}
                override fun onPartialResults(partialResults: Bundle?) {}
                override fun onEvent(eventType: Int, params: Bundle?) {}
            }
        )
    }

    private fun startListening() {

        listenButton.isEnabled = false

        val intent = Intent(
            RecognizerIntent.ACTION_RECOGNIZE_SPEECH
        )

        intent.putExtra(
            RecognizerIntent.EXTRA_LANGUAGE_MODEL,
            RecognizerIntent.LANGUAGE_MODEL_FREE_FORM
        )

        intent.putExtra(
            RecognizerIntent.EXTRA_LANGUAGE,
            "hi-IN"
        )

        intent.putExtra(
            RecognizerIntent.EXTRA_PROMPT,
            "JARVIS सुन रहा है"
        )

        speechRecognizer.startListening(intent)
    }

    private fun speak(message: String) {

        statusText.text = "JARVIS: $message"

        textToSpeech.speak(
            message,
            TextToSpeech.QUEUE_FLUSH,
            null,
            "JARVIS"
        )

        listenButton.isEnabled = true
    }

    private fun runCommand(command: String) {

        when {

            command.contains("नमस्ते") ||
            command.contains("hello") ||
            command.contains("हेलो") -> {

                speak("नमस्ते! मैं JARVIS हूँ।")
            }

            command.contains("youtube") ||
            command.contains("यूट्यूब") -> {

                openWebsite(
                    "https://www.youtube.com",
                    "YouTube खोल रहा हूँ"
                )
            }

            command.contains("google") ||
            command.contains("गूगल") -> {

                openWebsite(
                    "https://www.google.com",
                    "Google खोल रहा हूँ"
                )
            }

            command.contains("समय") ||
            command.contains("टाइम") ||
            command.contains("time") -> {

                val time = SimpleDateFormat(
                    "hh:mm a",
                    Locale.getDefault()
                ).format(Date())

                speak("अभी समय है $time")
            }

            command.contains("क्रोम") ||
            command.contains("chrome") -> {

                val chrome =
                    packageManager.getLaunchIntentForPackage(
                        "com.android.chrome"
                    )

                if (chrome != null) {

                    startActivity(chrome)
                    speak("Chrome खोल रहा हूँ")

                } else {

                    speak("Chrome फोन में नहीं मिला")
                }
            }

            command.contains("गूगल सर्च") -> {

                openWebsite(
                    "https://www.google.com",
                    "Google खोल रहा हूँ"
                )
            }

            else -> {

                speak(
                    "माफ कीजिए, यह command अभी मुझे नहीं आती"
                )
            }
        }
    }

    private fun openWebsite(
        url: String,
        message: String
    ) {

        val intent = Intent(
            Intent.ACTION_VIEW,
            Uri.parse(url)
        )

        startActivity(intent)

        speak(message)
    }

    override fun onInit(status: Int) {

        if (status == TextToSpeech.SUCCESS) {

            textToSpeech.language =
                Locale("hi", "IN")
        }
    }

    override fun onDestroy() {

        speechRecognizer.destroy()
        textToSpeech.shutdown()

        super.onDestroy()
    }
}

<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission
        android:name="android.permission.RECORD_AUDIO"/>

    <application
        android:theme="@style/AppTheme"
        android:label="JARVIS">

        <activity
            android:name=".MainActivity"
            android:exported="true">

            <intent-filter>

                <action
                    android:name="android.intent.action.MAIN"/>

                <category
                    android:name="android.intent.category.LAUNCHER"/>

            </intent-filter>

        </activity>

    </application>

</manifest>
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
}

android {

    namespace 'com.example.jarvis'

    compileSdk 35

    defaultConfig {

        applicationId "com.example.jarvis"

        minSdk 26

        targetSdk 35

        versionCode 1

        versionName "1.0"
    }
}pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {

    repositoriesMode.set(
        RepositoriesMode.FAIL_ON_PROJECT_REPOS
    )

    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "JARVIS"

include(":app")
