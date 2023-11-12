# Proof of Concept - Rolling Codes in FOB for Educational Purpose

## Idea
The project aims to create a basic Python-based rolling code system, serving as an educational tool to demonstrate the functionality of remote devices, such as car unlock systems. This implementation is simplified and not intended for real-world use but offers a clear, conceptual understanding of the technology.

## Notes
This is neither a secure nor a functional solution for practical applications. The development is loosely inspired by BMW's E-Series mechanics and a simplified version of the KeeLoq system. To aid understanding, certain complexities are reduced. For instance, data storage and transmission are handled using JSON payloads, as opposed to bit-level operations typical in actual devices.

## How it works

<b>Key Points:</b>
- The sender (key fob) only transmits data and does not receive any. This limits the encryption strength.
- Typically, the sender lacks real-time clocks, pseudorandom number generators, and other features necessary for robust encryption.

<b>Workflow:</b>
1. Pairing: The fob pairs with the receiver (e.g., a car) usually initiated by a button sequence on the receiver. The fob sends its ID and a seed value.
2. Code Generation: Both the sender and receiver generate a list of rolling codes using the sender's public ID and the private seed. This allows for different, synchronized code lists for each sender, enabling multiple fobs per receiver.
3. Transmission: When activated, the fob sends a payload containing its ID, a code from the list (selected based on an internal counter), and sometimes the button state. It then increments its counter and generates a new code to ensure an ongoing supply.
4. Reception and Verification: The receiver processes the payload, using its own counter to retrieve the corresponding code. If the codes match, the receiver increments its counter.
5. Sync Issues: If a fob is used out of the receiver's range, its counter advances more than the receiver's, causing a mismatch. The receiver has a threshold for checking subsequent codes but never checks past codes. If a match is found within this threshold, it re-syncs by adjusting its counter. Otherwise, re-pairing is necessary.


## Possible Attacks

### Replay Attack
An attacker intercepts (man-in-the-middle attack) and saves a rolling code, attempting to reuse it for unauthorized access. This is generally thwarted by the ever-increasing counter, which doesn't decrease. However, some systems remain vulnerable, as evidenced in certain cases (e.g., Honda key fob vulnerability, detailed here: https://securityintelligence.com/articles/what-to-know-honda-key-fob-vulnerability/).

### Rolljam Attack
This more sophisticated attack involves jamming and recording the first transmission, misleading the user to retransmit. The attacker then uses the first code immediately and reserves the second for future unauthorized access. More details are available in Samy Kamkar's DEF CON presentation (https://samy.pl/defcon2015/).

<b>Workflow:</b>
1. A user activates the fob button, initiating the first signal. An attacker then disrupts and captures this signal, leading to no transmission to and no response from the receiver.
2. Assuming a malfunction or range issue with the fob, the user typically attempts to use the fob again. During this second attempt, the attacker also interferes and records the signal, thus obtaining two valid rolling codes.
3. The attacker promptly transmits the first recorded code to the receiver. The receiver responds this time, leading the user to believe the second attempt was successful and the initial failure was a mere glitch.
4. Unbeknownst to the user, the attacker retains the second valid code, which remains effective for unauthorized access later, provided there are no subsequent transmissions from the fob.

<b>Technical Explaination:</b>
From a technical standpoint, suppose the receiver's counter is at a value like 61. When the user presses the fob button, the fob transmits codes 61 and 62 successively, but both signals are intercepted by the attacker. The attacker then sends code 61 to the receiver, which the receiver accepts as valid, advancing its counter to 62. As a result, the attacker retains the code 62, which remains valid for unauthorized use since the receiver's counter matches or is less than 62, due to the earlier signal interference.


## Demo cases in this repository
1. Pairing multiple devices with different seeds.
2. Sending a valid request (e.g., pressing the fob button).
3. Sending a valid request with an out-of-sync counter.
4. Sending an invalid request due to significant counter mismatch.
5. Replay attack demonstration.
6. Rolljam attack scenario.


## Resources
For more detailed technical information, refer to the KeeLoq documentation available: https://ww1.microchip.com/downloads/en/AppNotes/91002a.pdf