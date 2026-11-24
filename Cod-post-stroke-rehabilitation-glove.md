# post-stroke-rehabilitation-glove.
Bachelor’s thesis project featuring a sensor-based glove for post-stroke hand rehabilitation. The Arduino prototype tracks hand and finger movements and displays sensor readings in a Python interface.
- An Arduino Uno R3–based glove that tracks finger bending and hand motion.
- Pulse rate and SpO₂ monitoring using a GY-MAX30100 pulse oximeter.
- A Python interface that receives and visualizes sensor readings over a serial connection.
- An exploratory Random Forest analysis on a 30-patient dataset, with classification metrics and feature importance plots.

## Prototype

This is an academic prototype developed for my 2025 bachelor’s thesis in Medical Engineering at Politehnica Bucharest.

## Arduino Code 

#include <Wire.h>
#include "MAX30100_PulseOximeter.h"
#include <MPU6050.h>

#define REPORTING_PERIOD_MS 1000

PulseOximeter pox;
MPU6050 mpu;

uint32_t tsLastReport = 0;

void onBeatDetected() {
    Serial.println("Bataie detectata!");
}

void setup() {
    Serial.begin(9600);
    Wire.begin();

    if (!pox.begin()) {
        Serial.println("Eroare MAX30100");
        while (1);
    }
    pox.setOnBeatDetectedCallback(onBeatDetected);

    mpu.initialize();
    if (!mpu.testConnection()) {
        Serial.println("Eroare MPU-6500");
        while (1);
    }
}

void loop() {
    pox.update();

    if (millis() - tsLastReport > REPORTING_PERIOD_MS) {
        tsLastReport = millis();

        float bpm = pox.getHeartRate();
        float spo2 = pox.getSpO2();

        int16_t ax, ay, az, gx, gy, gz;
        mpu.getMotion6(&ax, &ay, &az, &gx, &gy, &gz);

        Serial.print("BPM:"); Serial.print(bpm);
        Serial.print(", SpO2:"); Serial.print(spo2);
        Serial.print(", AX:"); Serial.print(ax);
        Serial.print(", AY:"); Serial.print(ay);
        Serial.print(", AZ:"); Serial.print(az);
        Serial.print(", GX:"); Serial.print(gx);
        Serial.print(", GY:"); Serial.print(gy);

## Python code
import pandas as pd
import numpy as np
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, classification_report
from sklearn.preprocessing import StandardScaler
import matplotlib.pyplot as plt


# Citire date
train_data = pd.read_csv('./antrenare/pacienti_date_antrenare.csv').iloc[:, 1:].values
test_data = pd.read_csv('./testare/pacienti_date_test.csv').iloc[:, 1:].values

train_labels = pd.read_csv('./antrenare/etichete_pacienti_antrenare.csv').iloc[:, 1].values
test_labels = pd.read_csv('./testare/etichete_pacienti_test.csv').iloc[:, 1].values


# Scalare date
scaler = StandardScaler()
train_data_scaled = scaler.fit_transform(train_data)
test_data_scaled = scaler.transform(test_data)


# Antrenare model
clf = RandomForestClassifier()
clf.fit(train_data_scaled, train_labels)

np.random.seed(123)
test_data = test_data_scaled + np.random.normal(0, 0.3, size=test_data_scaled.shape)


# Predictii
predictions = clf.predict(test_data)


# Evaluare
accuracy = accuracy_score(test_labels, predictions)
print(f"Acuratețe: {accuracy:.2f}")
print(classification_report(test_labels, predictions))


# Output în fișier
with open('output.txt', 'w') as f:
    f.write("=== train_data ===\n")
    for i, (row, label) in enumerate(zip(train_data, train_labels)):
        f.write(f"Pacient {i+1}: {row.tolist()} cu eticheta {label}\n")

    f.write("\n=== test_data ===\n")
    for i, row in enumerate(test_data):
        f.write(f"Pacient {i+1}: {row.tolist()}\n")

    f.write("\n=== Predictii vs Etichete reale ===\n")
    f.write("Predictii: " + ' '.join(map(str, predictions)) + "\n")
    f.write("Etichete: " + ' '.join(map(str, test_labels)) + "\n")


# Grafic 1
feature_importances = clf.feature_importances_
feature_names = [f'F{i+1}' for i in range(train_data.shape[1])]

plt.figure(figsize=(10, 6))
percent_importances = feature_importances * 100
bars = plt.bar(
    range(len(percent_importances)),
    percent_importances,
    tick_label=feature_names,
    color='turquoise'
)

for bar, val in zip(bars, percent_importances):
    plt.text(
        bar.get_x() + bar.get_width() / 2,
        bar.get_height() + 1,
        f'{val:.1f}%',
        ha='center',
        va='bottom'
    )

plt.xticks(rotation=45)
plt.title("Importanța caracteristicilor (%) - Random Forest")
plt.xlabel("Caracteristici")
plt.ylabel("Importanță (%)")
plt.ylim(0, max(percent_importances) + 5)
plt.tight_layout()
plt.show()


# Grafic 2
accuracy_percent = accuracy * 100
correct = predictions == test_labels
colors = ['green' if c else 'red' for c in correct]

plt.figure(figsize=(12, 2.5))
plt.bar(np.arange(len(test_labels)), [1] * len(test_labels), color=colors, edgecolor='black')

plt.title(f"Corectitudine predicții model – Acuratețe: {accuracy_percent:.2f}%", fontsize=12)
plt.xlabel("Index pacient")
plt.yticks([])
plt.tight_layout()
plt.show()
        Serial.print(", GZ:"); Serial.println(gz);
    }
}
