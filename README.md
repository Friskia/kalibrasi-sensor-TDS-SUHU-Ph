# kalibrasi-sensor-TDS-SUHU-Ph
kalibrasi sensor TDS, SUHU, Ph
#include <OneWire.h>
#include <DallasTemperature.h>

// =====================================
// PIN SENSOR
// =====================================

#define PH_PIN         34
#define TDS_PIN        35
#define ONE_WIRE_BUS   4

// =====================================
// DS18B20
// =====================================

OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);

// =====================================
// VARIABLE
// =====================================

float voltagePH;
float voltageTDS;

// =====================================
// FILTER PEMBACAAN ADC
// =====================================

float readAverageADC(int pin, int jumlahSample) {

  long total = 0;

  for (int i = 0; i < jumlahSample; i++) {

    total += analogRead(pin);

    delay(10);
  }

  return total / (float)jumlahSample;
}

// =====================================
// SETUP
// =====================================

void setup() {

  Serial.begin(115200);

  // Resolusi ADC ESP32
  analogReadResolution(12);

  // Range ADC lebih stabil
  analogSetAttenuation(ADC_11db);

  sensors.begin();

  Serial.println();
  Serial.println("===== SISTEM MONITORING AIR =====");
}

// =====================================
// LOOP
// =====================================

void loop() {

  // =====================================
  // BACA SUHU DS18B20
  // =====================================

  sensors.requestTemperatures();

  float temperature =
      sensors.getTempCByIndex(0);

  // =====================================
  // BACA SENSOR PH
  // =====================================

  float adcPH =
      readAverageADC(PH_PIN, 20);

  voltagePH =
      adcPH * (3.3 / 4095.0);

  // =====================================
  // RUMUS KALIBRASI PH
  // =====================================

  float ph =
      (7.40 * voltagePH)
      - 9.39;

  // =====================================
  // BACA SENSOR TDS
  // =====================================

  float adcTDS =
      readAverageADC(TDS_PIN, 20);

  voltageTDS =
      adcTDS * (3.3 / 4095.0);

  // =====================================
  // RUMUS KALIBRASI TDS
  // =====================================

  float tds =
      (398.23 * voltageTDS)
      + 42.24;

  // =====================================
  // BATASI NILAI MINIMUM
  // =====================================

  if (tds < 0) {
    tds = 0;
  }

  // =====================================
  // SERIAL MONITOR
  // =====================================

  Serial.println("===== MONITORING AIR =====");

  Serial.print("Suhu : ");
  Serial.print(temperature-0.8, 2);
  Serial.println(" C");

  Serial.print("pH   : ");
  Serial.println(ph, 2);

  Serial.print("TDS  : ");
  Serial.print(tds, 0);
  Serial.println(" ppm");

  Serial.print("Voltage pH  : ");
  Serial.println(voltagePH, 3);

  Serial.print("Voltage TDS : ");
  Serial.println(voltageTDS, 3);

  // =====================================
  // STATUS AIR
  // =====================================

  Serial.println();
  Serial.println("===== STATUS AIR =====");

  // Status pH

  if (ph < 6.5) {

    Serial.println("pH : ASAM");

  } else if (ph > 8.5) {

    Serial.println("pH : BASA");

  } else {

    Serial.println("pH : NORMAL");
  }

  // Status TDS

  if (tds < 150) {

    Serial.println("TDS : AIR BERSIH");

  } else if (tds < 300) {

    Serial.println("TDS : AIR SEDANG");

  } else {

    Serial.println("TDS : AIR KOTOR");
  }

  Serial.println();
  Serial.println("==============================");
  Serial.println();

  delay(2000);
}
