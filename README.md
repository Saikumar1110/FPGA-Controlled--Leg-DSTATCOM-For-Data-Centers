## FPGA-BASED REAL-TIME KALMAN FILTER STATE ESTIMATION AND CONTROL OF DSTATCOM
Summary
1. Nonlinear loads and the neutral problem.
Single-phase nonlinear loads, such as rectifiers in computers and chargers, produce triplen harmonics (3rd, 9th, 15th). These harmonics are in phase in all three phases, so they add up instead of cancelling and flow back through the neutral conductor, which can overload it.

2. The DSTATCOM as the remedy.
A DSTATCOM is a shunt converter connected at the point of common coupling. It injects a compensating current that cancels the harmonics, supplies reactive power, balances the load and handles the neutral current. This project uses three single-phase H-bridge converters, each connected to one phase through a transformer, and controls them with the synchronous reference frame method.

3. The reference current is the critical part.
The quality of compensation depends on how accurately the reference current is generated. The conventional SRF method gets its reference frame from a phase-locked loop. That loop is sensitive to distortion in the supply voltage, and it adds delay, which makes compensation lag the load.

4. What a Kalman filter does.
A Kalman filter is a recursive estimator. It combines a model of the signal with each new noisy measurement to produce the best estimate of the hidden quantities. Here, the hidden quantities are the in-phase and quadrature components of the fundamental and the odd harmonics of the load current. Once they are known, the reference current can be built directly from them.

5. Predict and correct.
At each sample, the filter first predicts the next state using its model, then corrects that prediction with the new measurement. The correction is weighted by the Kalman gain, which balances how much the filter trusts its model against how much it trusts the measurement. The noise levels assumed for the model and the measurement are set by tuning, and the filter's behaviour depends on them.

6. Why no PLL is needed.
The phase information is carried inside the state of the filter. The filter therefore does not need a separate synchronising loop, which removes the PLL's delay and its sensitivity to distorted voltage. Because the filter can also predict ahead, the delay from sampling and processing can be compensated.

7. Fixed-point arithmetic on the FPGA.
Floating-point arithmetic is expensive in FPGA logic, so the filter uses fixed-point numbers. The chosen format is Q15.16, a 32-bit word with 16 fractional bits, which gives a resolution of about 1.5 x 10^-5. Addition and subtraction saturate when they overflow. Multiplication uses a wider intermediate product, which is then scaled back to the same format.

8. Verilog building blocks.
The design starts with an adder, a subtractor and a multiplier. Each one is checked against the double-precision MATLAB result on a set of test inputs, and the maximum error is recorded. These blocks are the foundation for the matrix operations of the filter.

9. Pipelining and parallel processing.
A DSP processor runs operations one after another, so the whole filter update takes most of the sampling period. An FPGA can instead run many operations at once, and pipelining splits each operation into stages so that a new input can enter on every clock cycle once the pipeline is full. The matrix products and the matrix inversion are the most costly steps, so the plan is a folded, systolic-style array that reuses multiply-add units. The trade-off is that more parallelism uses more logic and DSP blocks, and the number of available DSP blocks limits how many states the filter can have.

10. Target and verification.
The design will be implemented on a Digilent Nexys 7 board, which has an Artix-7 FPGA and a 100 MHz clock. The timing and resource reports will show whether one filter update fits within one sampling period. The Kalman reference will then be compared with the SRF method under load steps and distorted supply voltage, and the speed, power and reliability of the FPGA design will be reported together.



