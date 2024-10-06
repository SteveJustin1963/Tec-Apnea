



```cpp
#include <Arduino.h>
#include <arduinoFFT.h>

#define SAMPLES 128             // Must be a power of 2
#define SAMPLING_FREQUENCY 1000 // Hz, must be less than 10000 due to ADC

unsigned int sampling_period_us;
unsigned long microseconds;

double vReal[SAMPLES];
double vImag[SAMPLES];

arduinoFFT FFT = arduinoFFT();

// Pin definitions
const int SOUND_SENSOR_PIN = A0;  // Analog input for sound sensor
const int BUZZER_PIN = 9;         // Digital output for buzzer

// Thresholds and constants
const int APNEA_THRESHOLD = 500;     // Adjust based on your sensor's sensitivity
const int HYPOPNEA_THRESHOLD = 300;  // Adjust based on your sensor's sensitivity
const int RERA_THRESHOLD = 700;      // Adjust based on your sensor's sensitivity
const int MAX_EVENTS = 10;
const int ALARM_DURATION = 5000;     // 5 seconds in milliseconds

// Frequency thresholds for snoring (adjust as needed)
const int SNORING_FREQ_LOW = 60;   // Hz
const int SNORING_FREQ_HIGH = 300; // Hz

// Variables to track events
int apneaCount = 0;
int hypopneaCount = 0;
int reraCount = 0;

void setup() {
  Serial.begin(115200);
  pinMode(SOUND_SENSOR_PIN, INPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  sampling_period_us = round(1000000 * (1.0 / SAMPLING_FREQUENCY));
  Serial.println("Sleep Apnea Monitor with FFT Started");
}

void loop() {
  // Sampling
  for (int i = 0; i < SAMPLES; i++) {
    microseconds = micros();
    vReal[i] = analogRead(SOUND_SENSOR_PIN);
    vImag[i] = 0;
    while (micros() < (microseconds + sampling_period_us)) {
      // wait
    }
  }

  // FFT
  FFT.Windowing(vReal, SAMPLES, FFT_WIN_TYP_HAMMING, FFT_FORWARD);
  FFT.Compute(vReal, vImag, SAMPLES, FFT_FORWARD);
  FFT.ComplexToMagnitude(vReal, vImag, SAMPLES);

  // Find dominant frequency
  double peak = FFT.MajorPeak(vReal, SAMPLES, SAMPLING_FREQUENCY);

  // Check if the dominant frequency is in the snoring range
  if (peak >= SNORING_FREQ_LOW && peak <= SNORING_FREQ_HIGH) {
    Serial.print("Snoring detected at frequency: ");
    Serial.print(peak);
    Serial.println(" Hz");

    // Determine the type of event based on amplitude
    int soundLevel = vReal[0]; // Use the DC component as overall amplitude
    if (soundLevel < HYPOPNEA_THRESHOLD) {
      hypopneaCount++;
      Serial.println("Hypopnea event detected");
    } else if (soundLevel < APNEA_THRESHOLD) {
      apneaCount++;
      Serial.println("Apnea event detected");
    } else if (soundLevel > RERA_THRESHOLD) {
      reraCount++;
      Serial.println("RERA event detected");
    }
  } else {
    Serial.println("No snoring detected");
  }

  int totalEvents = apneaCount + hypopneaCount + reraCount;
  
  if (totalEvents >= MAX_EVENTS) {
    triggerAlarm();
    resetCounts();
  }

  delay(1000); // Wait for a second before the next sample
}

void triggerAlarm() {
  Serial.println("ALARM TRIGGERED!");
  tone(BUZZER_PIN, 1000); // 1kHz tone
  delay(ALARM_DURATION);
  noTone(BUZZER_PIN);
  Serial.println("Alarm stopped");
}

void resetCounts() {
  apneaCount = 0;
  hypopneaCount = 0;
  reraCount = 0;
  Serial.println("Event counts reset");
}

```

This updated code introduces several new elements:

1. **FFT Library**: We're using the ArduinoFFT library to perform frequency analysis.

2. **Sampling**: We collect 128 samples at a 1kHz sampling rate. These parameters can be adjusted based on your specific needs and the capabilities of your Arduino.

3. **Frequency Analysis**: The FFT is computed on the collected samples, and we find the dominant frequency using the `MajorPeak` function.

4. **Snoring Detection**: We check if the dominant frequency falls within a specified range (60-300 Hz in this example) that is typical for snoring sounds. You may need to adjust this range based on actual measurements and research.

5. **Event Classification**: If snoring is detected, we then use the amplitude (represented by the DC component of the FFT) to classify the event as apnea, hypopnea, or RERA.

To use this code:

1. Install the ArduinoFFT library in your Arduino IDE (Sketch -> Include Library -> Manage Libraries, then search for "ArduinoFFT" and install it).

2. Upload the code to your Arduino Uno.

3. Ensure your connections are correct:
   - Sound sensor to A0
   - Buzzer to pin 9

4. Open the Serial Monitor (set to 115200 baud) to see the output.

Remember to calibrate the frequency range and amplitude thresholds based on your specific sensor and real-world testing. This implementation provides a basic framework, but for a truly accurate sleep apnea monitor, you would need:

1. More sophisticated signal processing techniques
2. Machine learning algorithms for better classification
3. Clinical validation of the results

This code serves as a starting point and educational tool rather than a medical-grade device. Always consult with healthcare professionals for actual sleep apnea diagnosis and treatment.
