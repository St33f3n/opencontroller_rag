# Implementierung von CRSF-Paketen in Rust für RC-Anwendungen

Dieses Dokument bietet einen detaillierten Überblick über die Implementierung von CRSF (Crossfire Serial Protocol)-Paketen in Rust, um RC-Kanalwerte und Telemetrie-Daten zu übertragen. Die Erläuterungen basieren auf den offiziellen CRSF-Dokumentationen, die unter citegithub-crsf-wg und citedocs-crsf-rs verfügbar sind.

## Überblick über das CRSF-Protokoll

CRSF ist ein speziell entwickeltes serielles Protokoll, das in RC-Systemen eingesetzt wird, um Befehle (z. B. Steuerkanäle) sowie Telemetrie-Daten (z. B. RSSI, Batteriestatus) zwischen Sender und Empfänger zu übertragen. Durch die Nutzung einer stabilen UART-Verbindung (Universal Asynchronous Receiver-Transmitter) gewährleistet CRSF eine zuverlässige und effiziente Kommunikation, die besonders in zeitkritischen Anwendungen von großer Bedeutung ist.

## CRSF-Paketstruktur

Ein CRSF-Paket besteht aus mehreren definierten Komponenten:

- **Header:** Das Paket beginnt mit einem Startbyte, das gefolgt von Informationen zur Paketlänge kommt. Dies ermöglicht dem Empfänger, die Grenzen eines Pakets im kontinuierlichen Datenstrom zu erkennen.
- **Pakettyp:** Dieser Teil des Pakets spezifiziert, welche Art von Daten übertragen werden. Typische Beispiele sind RC-Kanäle, Telemetrie oder Befehle.
- **Payload (Nutzdaten):** Bei RC-Kanalpaketen enthält die Nutzlast in der Regel 16-Bit-Werte, die die Steuerpositionen repräsentieren. Die genaue Anordnung und Interpretation der Daten ist im CRSF-Spezifikationsdokument festgelegt.
- **CRC (Checksum):** Um die Datenintegrität zu überprüfen, wird dem Paket eine Prüfsumme (häufig CRC8) hinzugefügt. Der Empfänger vergleicht diese Prüfsumme mit einer eigenen Berechnung, um Übertragungsfehler zu erkennen.

Diese strukturierte Herangehensweise ermöglicht eine fehlerresistente und eindeutige Datenübertragung zwischen den Systemkomponenten citegithub-crsf-wg, citedocs-crsf-rs.

## Aufbau eines RC-Kanal-Pakets in Rust

Das [crsf Rust-Crate](https://docs.rs/crsf/latest/crsf/index.html) bietet eine komfortable API, um CRSF-Pakete zu erstellen und zu verarbeiten. Nachfolgend ein Beispielcode, der den Aufbau und Versand eines RC-Kanal-Pakets demonstriert:

```rust
use crsf::packet::{CrsfPacket, PacketType};
use serialport::SerialPort;
use std::time::Duration;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Beispiel: 16 RC-Kanäle, alle auf den Neutralwert 1500 gesetzt
    let rc_channels: [u16; 16] = [1500; 16];

    // Erstellen eines CRSF-Pakets für RC-Kanäle
    let packet = CrsfPacket::new_rc_channels(&rc_channels);
    // Intern wird das Paket gemäß CRSF-Struktur (Header, Typ, Payload, CRC) aufgebaut.
    let packet_bytes = packet.serialize();

    // Öffnen der seriellen Schnittstelle (z. B. "/dev/ttyUSB0" unter Linux, "COM3" unter Windows)
    let port_name = "/dev/ttyUSB0";
    // Die Baudrate muss exakt den Vorgaben des CRSF-Protokolls entsprechen (hier 420.000 Baud)
    let baud_rate = 420_000;
    let mut port = serialport::new(port_name, baud_rate)
        .timeout(Duration::from_millis(10))
        .open()?;

    // Versand des CRSF-Pakets über die serielle Schnittstelle
    port.write_all(&packet_bytes)?;
    println!("RC-Kanalpaket gesendet: {:?}", packet_bytes);

    Ok(())
}
```

In diesem Beispiel übernimmt die Funktion `CrsfPacket::new_rc_channels` den korrekten Aufbau des Pakets. Die Serialisierung wandelt das Paket in ein Byte-Array um, das anschließend über die serielle Verbindung versendet wird. Die exakte Einstellung der Baudrate und Timeouts ist dabei von zentraler Bedeutung, um die fehlerfreie Kommunikation sicherzustellen.

## Befehle senden und Telemetrie empfangen

Neben der Übertragung von RC-Kanalwerten unterstützt CRSF auch den Versand von Befehlen und das Empfangen von Telemetrie-Daten. 

### Senden von Befehlen

Befehle werden über speziell definierte Pakettypen (z. B. `PacketType::Command`) gesendet. Der Aufbau eines Befehls-Pakets ähnelt dem des RC-Kanalpakets. Hierbei wird der entsprechende Pakettyp gewählt, und die Payload enthält die spezifischen Befehlsdaten. Diese Befehle können etwa für Bind-Prozeduren oder Konfigurationsänderungen genutzt werden.

### Empfang von Telemetrie-Daten

Telemetrie-Daten werden als Rückmeldungen vom Empfänger (z. B. ein ExpressLRS-Sender) übermittelt. Zur Verarbeitung dieser Daten ist es notwendig, die serielle Schnittstelle kontinuierlich auszulesen und den eintreffenden Byte-Strom korrekt zu parsen. Ein Beispielcode verdeutlicht diesen Prozess:

```rust
use crsf::packet::CrsfPacket;
use serialport::SerialPort;
use std::time::Duration;
use std::io::Read;

fn receive_telemetry(port: &mut dyn SerialPort) -> Result<(), Box<dyn std::error::Error>> {
    let mut buf = [0u8; 256];
    loop {
        let n = port.read(&mut buf)?;
        if n > 0 {
            // Der Parser versucht, das empfangene Byte-Array in ein CRSF-Paket zu konvertieren.
            if let Ok(packet) = CrsfPacket::parse(&buf[..n]) {
                match packet.packet_type() {
                    PacketType::Telemetry => {
                        println!("Telemetrie empfangen: {:?}", packet.payload());
                    }
                    _ => {
                        // Verarbeitung anderer Pakettypen
                    }
                }
            }
        }
    }
}
```

Hierbei wird besonders auf die Erkennung der Paketgrenzen geachtet, da die serielle Schnittstelle einen kontinuierlichen Datenstrom liefert. Die Validierung mittels CRC ist essenziell, um fehlerhafte Pakete auszuschließen. Je nach Anwendungsfall kann es sinnvoll sein, asynchrone oder blockierende Lesevorgänge zu implementieren, um einen reibungslosen Kommunikationsfluss zu gewährleisten.

## Zusammenfassung

In diesem Dokument wurden die wesentlichen Aspekte der Implementierung von CRSF-Paketen in Rust detailliert erläutert. Angefangen bei der Definition der CRSF-Paketstruktur – bestehend aus Header, Pakettyp, Payload und CRC – bis hin zur praktischen Umsetzung des Versands von RC-Kanalwerten und dem Empfang von Telemetrie-Daten. Das bereitgestellte Rust-Codebeispiel demonstriert, wie mithilfe des crsf-Crates Pakete korrekt aufgebaut, serialisiert und über eine serielle Schnittstelle übertragen werden können. Ebenso wurde beschrieben, wie Befehle gesendet werden und Telemetrie-Daten durch kontinuierliches Lesen und Parsen des Datenstroms verarbeitet werden. Diese Implementierungsdetails bieten eine solide Grundlage für die Entwicklung robuster und zuverlässiger RC-Anwendungen.  
  
citegithub-crsf-wg  
citedocs-crsf-rs
