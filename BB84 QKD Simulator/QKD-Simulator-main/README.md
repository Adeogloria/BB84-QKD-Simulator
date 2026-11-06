# Quantum Key Distribution (QKD) Simulator 🛡️🔐

![QKDSimulator]

A Python simulator of the **BB84 quantum key distribution protocol**, built with Cirq and Tkinter. It is the reference implementation for my final-year dissertation, **"[Dissertation title]"**, submitted for the B.Sc. in Management Information Systems at Covenant University ([2026]).

The GUI lets users type a message, choose a channel scenario (normal, noisy or eavesdropped), and watch the key exchange succeed or collapse. When it succeeds, the message is encrypted with the quantum-derived key and decrypted again. Two built-in experiments show how channel noise and fiber distance affect the key.

## Research Background

The project investigates whether BB84 can be simulated realistically enough to:

1. Show that an intercept-and-resend attack pushes the QBER well above the 11% security threshold
2. Quantify how channel noise (0–25%) affects the proceed/abort decision
3. Measure how fiber distance (10–200 km) reduces key length through photon loss and detector inefficiency

| Experiment | Parameters | Expected result |
|---|---|---|
| Eavesdropping | Intercept-and-resend on all qubits | QBER ≈ 25% (theoretical), abort |
| Noise sweep | 0%, 2%, 5% … 25% flip rate | Abort once QBER > 11% |
| Distance | 10, 50, 100, 200 km at 0.2 dB/km | Key length falls with distance |


## Concepts Used

- **Quantum Key Distribution (QKD)**, a secure communication method that uses the properties of quantum mechanics to establish a shared secret key between two parties.
- **Quantum Gates**, operations performed on qubits to manipulate their states. This project uses the Pauli-X gate to encode a bit value of 1 and the Hadamard gate (H) to switch to the X basis.
- **Quantum Measurement**, which extracts classical bits from qubits. Measuring collapses the superposition, so an eavesdropper who measures in the wrong basis disturbs the state.
- **Classical Communication**, the public exchange of basis choices between Alice and Bob so they can sift the key.
- **QBER (Quantum Bit Error Rate)**, the error rate in a sample of the sifted key. Above 11%, the protocol aborts because the channel is too noisy or an eavesdropper is present.
- **Error Correction**, which reconciles differences between Alice's and Bob's keys. Currently a placeholder (see Limitations).
- **Privacy Amplification**, which distills a shorter, more secure key from the sifted key using a SHA-256 hash.
- **Authenticated Encryption (AES-256-GCM)**, which encrypts and authenticates the user's message with the final key.

## Technologies Used

- **cirq** for quantum circuit simulation
- **tkinter** for the GUI
- **cryptography** for AES-GCM encryption
- **secrets** for cryptographically secure random bits and bases
- **hashlib** and **hmac** for SHA-256 hashing and message authentication helpers
- **threading** to keep the GUI responsive during simulations

## Features

- Three scenarios selectable in the GUI: `normal`, `noisy` (5% bit-flip rate) and `eavesdrop`
- Eavesdropper collapse report showing intercepted qubits, corrupted bits and QBER
- **Noise sweep** across 0–25% noise, shown in a results table
- **Distance experiment** modelling fiber loss (0.2 dB/km), photon loss, detector inefficiency (85%) and channel noise
- Live display of the QBER, sifted key length and the encrypted and decrypted message

## How Does It Work?

1) **Initialisation.** The `QKDSimulator` class sets up the Cirq simulator, the qubits and the Tkinter GUI.
2) **Key Generation.** The user enters a message and chooses a scenario. Alice generates random bits and random bases (Z or X) using `secure_bits`.
3) **Qubit Preparation.** `alice_prepare` encodes each bit as a qubit: an X gate for a 1, then a Hadamard gate if the basis is X.
4) **Transmission.** Depending on the scenario, the qubits pass through an ideal channel, a noisy channel (`apply_noise`), or an intercept-and-resend attack by Eve who measures in random bases and re-sends.
5) **Measurement.** Bob measures each qubit in a randomly chosen basis using `bob_measure`.
6) **Key Sifting.** Alice and Bob publicly compare bases and keep only the positions where they match (`sift_key`).
7) **Eavesdropping Detection.** `estimate_qber` compares a sample of the sifted keys. If the QBER exceeds 11%, the transmission is aborted and a collapse report is shown for eavesdropping.
8) **Error Correction and Privacy Amplification.** The sifted key passes through `error_correction` and `privacy_amplification` (SHA-256) to produce the final key.
9) **Message Encryption and Decryption.** The final key encrypts the message with AES-GCM (`encrypt_message_aes`) and decrypts it again (`decrypt_message_aes`).
10) **Output Display.** The ciphertext, decrypted message, QBER and verdict are displayed in the GUI.

## Installation

```bash
git clone https://github.com/Adeogloria/QKD-Simulator.git
cd QKD-Simulator
pip install cirq cryptography
```

Requires Python 3.9+. Tkinter ships with most Python installs (on Linux: `sudo apt install python3-tk`).

## Usage

```bash
python QKDSimulator.py
```

1. Choose a scenario from the dropdown
2. Type a message and press Enter
3. Read the QBER, sifted key length, verdict, and the encrypted and decrypted message
4. Click **Run Noise Sweep** or **Run Distance Experiment** for the results tables

## Project Structure

| Component | Purpose |
|---|---|
| `alice_prepare`, `bob_measure` | Cirq circuits for encoding and measurement |
| `sift_key`, `estimate_qber` | Sifting and error estimation |
| `apply_noise`, `apply_photon_loss`, `apply_detector_inefficiency` | Channel imperfections |
| `distance_to_loss_params` | Fiber distance to loss and noise conversion |
| `encrypt_message_aes`, `decrypt_message_aes` | AES-GCM encryption |
| `QKDSimulator` | Tkinter GUI and experiment runners |

## Limitations

This is an academic simulation, not production cryptography.

- Error correction is a placeholder; a real system would use Cascade or LDPC codes
- Privacy amplification uses a SHA-256 hash rather than a universal hash family sized to the leaked information
- Only intercept-and-resend attacks are modelled
- HMAC authentication helpers are included but not integrated into the protocol flow

## Potential Enhancements for the Future

- **Real error correction** (Cascade/LDPC) and proper privacy amplification
- **Advanced attacks and protections**, such as decoy-state BB84 and photon-number-splitting attacks
- **Visualisation and analysis**, such as plots of QBER vs noise and key rate vs distance
- **GUI integration** of the Eve interception-rate sweep (`run_eve_sweep`)
- **Practical applications**, such as secure messaging built on the simulated key exchange

## Citation

> Oluwajebe Adeogomitide Gloria. ([2026]). *Simulating Quantum-safe Cryptography: A Multi-Scenario Evaluation of the BB84 Quantum Key Distribution Protocol*. B.Sc. dissertation, Department of Computer and Information Sciences, Covenant University, Ota, Nigeria.


## References

- Adams, C., & Lloyd, S. (2019). *Understanding PKI: Concepts, standards, and deployment considerations* (2nd ed.). Addison-Wesley.
- Aleksandrowicz, G., Alexander, T., Barkoutsos, P., Bello, L., Ben-Haim, Y., Bucher, D., & Zoufal, C. (2019). *Qiskit: An open-source framework for quantum computing*. Zenodo. https://doi.org/10.5281/zenodo.2562111
- Bennett, C. H., & Brassard, G. (1984). Quantum cryptography: Public key distribution and coin tossing. *Proceedings of the IEEE International Conference on Computers, Systems and Signal Processing*, 175–179.
- Bennett, C. H., Brassard, G., Crépeau, C., & Maurer, U. M. (1995). Generalized privacy amplification. *IEEE Transactions on Information Theory, 41*(6), 1915–1923. https://doi.org/10.1109/18.476316
- Brassard, G., Lütkenhaus, N., Mor, T., & Sanders, B. C. (2000). Limitations on practical quantum cryptography. *Physical Review Letters, 85*(6), 1330–1333.
- Brassard, G., & Salvail, L. (1994). Secret-key reconciliation by public discussion. In T. Helleseth (Ed.), *Advances in Cryptology — EUROCRYPT '93* (Lecture Notes in Computer Science, Vol. 765, pp. 410–423). Springer.
- Chen, L., Jordan, S., Liu, Y. K., Moody, D., Peralta, R., Perlner, R., & Smith-Tone, D. (2016). *Report on post-quantum cryptography* (NISTIR 8105). National Institute of Standards and Technology. https://doi.org/10.6028/NIST.IR.8105
- Cirq Developers. (2021). *Cirq*. Zenodo. https://doi.org/10.5281/zenodo.4750446
- Csenkey, K., & Bindel, N. (2023). Post-quantum cryptographic assemblages and the governance of the quantum threat. *Information & Communications Technology Law, 32*(1), 1–26.
- Dahlberg, A., Skrzypczyk, M., Coopmans, T., Wubben, L., Rozpędek, F., Pompili, M., & Wehner, S. (2019). A link layer protocol for quantum networks. In *Proceedings of the ACM SIGCOMM* (pp. 159–173). ACM.
- Diamanti, E., Lo, H.-K., Qi, B., & Yuan, Z. (2016). Practical challenges in quantum key distribution. *NPJ Quantum Information, 2*, 16025. https://doi.org/10.1038/npjqi.2016.25
- Ekert, A. K. (1991). Quantum cryptography based on Bell's theorem. *Physical Review Letters, 67*(6), 661–663.
- Elkouss, D., Leverrier, A., Alléaume, R., & Boutros, J. J. (2009). Efficient reconciliation protocol for discrete-variable quantum key distribution. *Proceedings of the IEEE ISIT* (pp. 1879–1883).
- Fingerhuth, M., Babej, T., & Wittek, P. (2018). Open source software in quantum computing. *PLOS ONE, 13*(12), e0208561.
- Gisin, N., Ribordy, G., Tittel, W., & Zbinden, H. (2002). Quantum cryptography. *Reviews of Modern Physics, 74*(1), 145–195.
- Grover, L. K. (1996). A fast quantum mechanical algorithm for database search. In *Proceedings of the 28th Annual ACM STOC* (pp. 212–219).
- Hwang, W.-Y. (2003). Quantum key distribution with high loss: Toward global secure communication. *Physical Review Letters, 91*(5), 057901.
- Joshi, A., Bhalgat, P., Chavan, P., & Chaudhari, T. (2024). Guarding against quantum threats: A survey of post-quantum cryptography. *Cryptography, 8*(2), 20.
- Kadve, D. B., Kumar, B., & Prasad, S. B. (2025). Quantum cryptography and its implications for future cyber security trends. *International Journal of Innovative Technology and Exploring Engineering, 14*(3), 1–8.
- Kong, I., Janssen, M., & Bharosa, N. (2024). Realizing quantum-safe information sharing: Implementation and adoption challenges. *Government Information Quarterly, 41*(1), 101884.
- Kumar, M. (2022). Post-quantum cryptography: Algorithm standardization and performance analysis. *Array, 15*, 100242.
- Laudenbach, F., & Pacher, C. (2018). Continuous-variable quantum key distribution with Gaussian modulation. *Advanced Quantum Technologies, 1*(1), 1800011.
- Lo, H.-K., Curty, M., & Qi, B. (2012). Measurement-device-independent quantum key distribution. *Physical Review Letters, 108*(13), 130503.
- Lo, H.-K., Ma, X., & Chen, K. (2005). Decoy state quantum key distribution. *Physical Review Letters, 94*(23), 230504.
- Lydersen, L., Wiechers, C., Wittmann, C., Elser, D., Skaar, J., & Makarov, V. (2010). Hacking commercial quantum cryptography systems by tailored bright illumination. *Nature Photonics, 4*(10), 686–689.
- Mehic, M., Niemiec, M., Rass, S., Ma, J., Peev, M., Aguado, A., Martin, V., Schauer, S., Poppe, A., Pacher, C., & Voznak, M. (2020). Quantum key distribution: A networking perspective. *ACM Computing Surveys, 53*(5), Article 96.
- National Institute of Standards and Technology. (2001). *Advanced Encryption Standard (AES)* (FIPS PUB 197). U.S. Department of Commerce.
- National Institute of Standards and Technology. (2022). *Post-quantum cryptography standardization*. https://csrc.nist.gov/projects/post-quantum-cryptography
- Nielsen, M. A., & Chuang, I. L. (2000). *Quantum computation and quantum information*. Cambridge University Press.
- Peev, M., Pacher, C., Alléaume, R., Barreiro, C., Bouda, J., Boxleitner, W., & Zeilinger, A. (2009). The SECOQC quantum key distribution network in Vienna. *New Journal of Physics, 11*(7), 075001.
- Pirandola, S., Andersen, U. L., Banchi, L., Berta, M., Bunandar, D., Colbeck, R., & Wallden, P. (2020). Advances in quantum cryptography. *Advances in Optics and Photonics, 12*(4), 1012–1236.
- Renner, R. (2008). *Security of quantum key distribution* (Doctoral dissertation, ETH Zurich).
- Rivest, R. L., Shamir, A., & Adleman, L. (1978). A method for obtaining digital signatures and public-key cryptosystems. *Communications of the ACM, 21*(2), 120–126.
- Scarani, V., Bechmann-Pasquinucci, H., Cerf, N. J., Dušek, M., Lütkenhaus, N., & Peev, M. (2009). The security of practical quantum key distribution. *Reviews of Modern Physics, 81*(3), 1301–1350.
- Shannon, C. E. (1948). A mathematical theory of communication. *Bell System Technical Journal, 27*(3), 379–423.
- Shor, P. W. (1994). Algorithms for quantum computation: Discrete logarithms and factoring. *Proceedings of the 35th Annual Symposium on Foundations of Computer Science* (pp. 124–134).
- Shor, P. W., & Preskill, J. (2000). Simple proof of security of the BB84 quantum key distribution protocol. *Physical Review Letters, 85*(2), 441–444.
- Stebila, D., & Mosca, M. (2017). Post-quantum key exchange for the internet and the Open Quantum Safe project. In *Selected Areas in Cryptography – SAC 2016* (pp. 14–37). Springer.
- Vasquez, V. F. (2025). *Numerical simulation of quantum key distribution using the BB84 protocol* (Bachelor's thesis, Stockholm University).
- Wootters, W. K., & Zurek, W. H. (1982). A single quantum cannot be cloned. *Nature, 299*(5886), 802–803.

## Acknowledgements

Supervised by Dr Jemimah Nathaniel.

## Contributing

Contributions are welcome! Feel free to reach out with your suggestions, bug reports, or feature requests.

## License

This project is licensed under the [MIT License](LICENSE).

## Author

Oluwajebe Adeogomitide Gloria · jebe.adeogo@gmail.com · https://www.linkedin.com/in/adeogomitide-gloria-a31a65298/?