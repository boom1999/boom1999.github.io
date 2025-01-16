---
title: F-OFDM vs OFDM
date: 2021-06-15
tags:
   - Modulation
   - Matlab
categories: Communication
---

:pushpin:

### Divide the OFDM cattier bandwidth into multiple sub bands with different parameters, and filter the sub bands, and try to leave as few isolation bands as possible between the sub bands

:sun_with_face::clock3::sleeping:

<!--more-->

---

## :books: Abstract

- *With the launch of the 3GPP 5G standard, the 5G sky has gradually disappeared, and the bright stars of the candidate technology have dazzled us, and the 5G serialized new air interface technology proposed by Huawei is undoubtedly the brightest star among them. F-OFDM can realize the slicing of the physical layer of the air interface and is backward compatible with the LTE 4G system, and can meet the needs of future 5G development.*

## :blue_book: OFDM vs F-OFDM

- *OFDM modulates high-rate data to mutually orthogonal sub-carriers through serial/parallel conversion, and introduces cyclic prefixes, which better solves the headache of inter-symbol interference. But the main problem of OFDM is not flexible enough.*
- *For example, the Internet of vehicles service with millisecond delay requires extremely short time-domain symbol and TTI; in the multi-connection scenario of the Internet of things, the amount of data transmitted by a single sensor is extremely low, but the requirements for the number of overall connections of the system are very high. It is necessary to configure a relatively narrow subcarrier spacing in the frequency domain. These flexible requirements cannot be met by OFDM technology.*
- *We can understand the time-frequency resources of the system as a carriage. If the OFDM scheme is adopted for decoration, the train can only provide fixed-size hard seats (sub-carrier spacing). Everyone, regardless of fat or thin, rich or not, can only sitting on a hard seat of the same size, this situation is obviously not humane enough. For 5G, we hope that the seats and spaces can be flexibly customized according to the height, short, fat and thin of passengers. Hard seats, soft seats, sleepers, boxes…you can adjust them whatever you wang. This the adaptive harmony train, and F-OFDM technology is based on this idea.*
- *F-OFDM can provide different sub-carrier spacing and numerology to meet the time-frequency resource requirements of different services. F-OFDM greatly reduces out-of-band leakage by optimizing the filter design, and the guard band overhead between different sub bands can be reduced to about 1%, which not only greatly improves the efficiency of spectrum utilization, but also, it’s possible to provides opportunities for future use of fragmented spectrum.*
- *In general, on the basis of inheriting all the advantages of OFDM (high spectrum utilization, adaptive MIMO, etc.), F-OFDM overcomes some inherent shortcomings of OFDM, and further improves flexibility and spectrum utilization efficiency basic technology to realize 5G air interface slicing.*

## :green_book: Benefits or advantages of F-OFDM

1. Efficient utilization of spectrum.
2. It suppresses out of band emission and hence guard band consumption can be reduced to minimum level.
3. Optimized numerology can be applied to suit the needs of certain type of services within each sub bands.
4. Possibility to incorporate other waveforms such as GFDM, FBMC, UFMC etc.
5. Backward and forward compatibility.
6. Using sub band-based filtering, global synchronization requirement is being relaxed. Hence inter-sub-band asynchronous transmission can be supported.

## :orange_book: Drawbacks or disadvantages of F-FDOM

1. The F-OFDM requires additional pair of transmit and receiver filters in transmitter and receiver chain respectively compare to conventional OFDM.
2. Like OFDM, CP can also be used in filter-based waveforms such as UFMC, F-OFDM to protect signal from SIS.
3. Since full-band filtering cannot remove any SIS, it exists in F-OFDM similar to standard OFDM.

## :open_book: Understanding of this example

- *This example compares Filtered-OFDM modulation with generic Cyclic Prefix OFDM (CP-OFDM) modulation. For F-OFDM, a well-designed filter is applied to the time domain OFDM symbol to improve the out-of-band radiation of the sub-band signal, while maintaining the complex-domain orthogonality of OFDM symbols. This example also models Filtered-OFDM modulation with configurable parameters. It highlights the filter design technique and the basic transmit/receive processing.*
- *A filter with a rectangular frequency response, i.e. a sinc impulse response, meets these criteria. To make this causal, the low-pass filter is realized using a window, which, effectively truncates the impulse response and offers smooth transitions to zero on both ends.*
- *In F-OFDM, the sub-band CP-OFDM signal is passed through the designed filter. As the filter's passband corresponds to the signal's bandwidth, only the few subcarriers close to the edge are affected. A key consideration is that the filter length can be allowed to exceed the cyclic prefix length for F-OFDM. The inter-symbol interference incurred is minimized due to the filter design using windowing (with soft truncation).*
- *Transmit-end processing operations are shown in the following F-OFDM transmitter diagram.*
- *The example next highlights the basic receive processing for F-OFDM for a single OFDM symbol. The received signal is passed through a matched filter, followed by the normal CP-OFDM receiver. It accounts for both the filtering ramp-up and latency prior to the FFT operation.*
- *No fading channel is considered in this example but noise is added to the received signal to achieve the desired SNR.*

## :closed_book: Conclusion

- *Comparing the plots of the spectral densities for CP-OFDM and F-OFDM schemes, F-OFDM has lower sidelobes. This allows a higher utilization of the allocated spectrum, leading to increased spectral efficiency.*
- *When we design F-OFDM filter, appropriate filtering for F-OFDM satisfies the following criteria:*

> 1. Should have a flat passband over the subcarriers in the sub-band.
> 2. Should have a sharp transition band to minimize guard-bands.
> 3. Should have sufficient stop-band attenuation.

- *Universal Filtered Multi-Carrier (UFMC) modulation scheme is another approach to sub-band filtered OFDM. For more information, see the UFMC vs. OFDM Modulation example. This F-OFDM example uses a single sub-band while the UFMC example uses multiple sub-bands.*
- *F-OFDM and UFMC both use time-domain filtering with subtle differences in the way the filter is designed and applied. For UFMC, the length of filter is constrained to be equal to the cyclic-prefix length, while for F-OFDM, it can exceed the CP length.*
- *For F-OFDM, the filter design leads to a slight loss in orthogonality (strictly speaking) which affects only the edge subcarriers.*

## :memo: Code

``` matlab
%% F-OFDM vs. OFDM Modulation
%

s = rng(211);       % Set RNG state for repeatability
 
%% System Parameters
%
% Define system parameters for the example. These parameters can be
% modified to explore their impact on the system.
 
numFFT = 1024;           % Number of FFT points
numRBs = 50;             % Number of resource blocks
rbSize = 12;             % Number of subcarriers per resource block
cpLen = 72;              % Cyclic prefix length in samples
 
bitsPerSubCarrier = 8;   % 2: QPSK, 4: 16QAM, 6: 64QAM, 8: 256QAM
snrdB = 18;              % SNR in dB
 
toneOffset = 2.5;        % Tone offset or excess bandwidth (in subcarriers)
L = 513;                 % Filter length (=filterOrder+1), odd
 
%% Filtered-OFDM Filter Design
%

numDataCarriers = numRBs*rbSize;    % number of data subcarriers in subband
halfFilt = floor(L/2);
n = -halfFilt:halfFilt;  
 
% Sinc function prototype filter
pb = sinc((numDataCarriers+2*toneOffset).*n./numFFT);
 
% Sinc truncation window
w = (0.5*(1+cos(2*pi.*n/(L-1)))).^0.6;
 
% Normalized lowpass filter coefficients
fnum = (pb.*w)/sum(pb.*w);
 
% Filter impulse response
h = fvtool(fnum, 'Analysis', 'impulse', ...
    'NormalizedFrequency', 'off', 'Fs', 15.36e6);
h.CurrentAxes.XLabel.String = 'Time (\mus)';
h.FigureToolbar = 'off';
 
% Use dsp filter objects for filtering
filtTx = dsp.FIRFilter('Structure', 'Direct form symmetric', ...
    'Numerator', fnum);
filtRx = clone(filtTx); % Matched filter for the Rx
 
% QAM Symbol mapper
qamMapper = comm.RectangularQAMModulator( ...
    'ModulationOrder', 2^bitsPerSubCarrier, 'BitInput', true, ...
    'NormalizationMethod', 'Average power');
 
%% F-OFDM Transmit Processing
%

% Set up a figure for spectrum plot
hFig = figure('Position', figposition([46 50 30 30]));
axis([-0.5 0.5 -200 -20]);
hold on; 
grid on
xlabel('Normalized frequency');
ylabel('PSD (dBW/Hz)')
title(['F-OFDM, ' num2str(numRBs) ' Resource blocks, '  ...
    num2str(rbSize) ' Subcarriers each'])
 
% Generate data symbols
bitsIn = randi([0 1], bitsPerSubCarrier*numDataCarriers, 1);
symbolsIn = qamMapper(bitsIn);
 
% Pack data into an OFDM symbol 
offset = (numFFT-numDataCarriers)/2; % for band center
symbolsInOFDM = [zeros(offset,1); symbolsIn; ...
                 zeros(numFFT-offset-numDataCarriers,1)];
ifftOut = ifft(ifftshift(symbolsInOFDM));
 
% Prepend cyclic prefix
txSigOFDM = [ifftOut(end-cpLen+1:end); ifftOut]; 
 
% Filter, with zero-padding to flush tail. Get the transmit signal
txSigFOFDM = filtTx([txSigOFDM; zeros(L-1,1)]);
 
% Plot power spectral density (PSD) 
[psd,f] = periodogram(txSigFOFDM, rectwin(length(txSigFOFDM)), ...
                      numFFT*2, 1, 'centered'); 
plot(f,10*log10(psd)); 
 
% Compute peak-to-average-power ratio (PAPR)
PAPR = comm.CCDF('PAPROutputPort', true, 'PowerUnits', 'dBW');
[~,~,paprFOFDM] = PAPR(txSigFOFDM);
disp(['Peak-to-Average-Power-Ratio for F-OFDM = ' num2str(paprFOFDM) ' dB']);
 
%% OFDM Modulation with Corresponding Parameters
%

% Plot power spectral density (PSD) for OFDM signal
[psd,f] = periodogram(txSigOFDM, rectwin(length(txSigOFDM)), numFFT*2, ...
                      1, 'centered'); 
hFig1 = figure('Position', figposition([46 15 30 30])); 
plot(f,10*log10(psd)); 
grid on
axis([-0.5 0.5 -100 -20]);
xlabel('Normalized frequency'); 
ylabel('PSD (dBW/Hz)')
title(['OFDM, ' num2str(numRBs*rbSize) ' Subcarriers'])
 
% Compute peak-to-average-power ratio (PAPR)
PAPR2 = comm.CCDF('PAPROutputPort', true, 'PowerUnits', 'dBW');
[~,~,paprOFDM] = PAPR2(txSigOFDM);
disp(['Peak-to-Average-Power-Ratio for OFDM = ' num2str(paprOFDM) ' dB']);

%% F-OFDM Receiver with No Channel
%

% Add WGN
rxSig = awgn(txSigFOFDM, snrdB, 'measured');
 
%%
% Receive processing operations are shown in the following F-OFDM receiver
% diagram.
%
 
% Receive matched filter
rxSigFilt = filtRx(rxSig);
 
% Account for filter delay 
rxSigFiltSync = rxSigFilt(L:end);
 
% Remove cyclic prefix
rxSymbol = rxSigFiltSync(cpLen+1:end);
 
% Perform FFT 
RxSymbols = fftshift(fft(rxSymbol));
 
% Select data subcarriers
dataRxSymbols = RxSymbols(offset+(1:numDataCarriers));
 
% Plot received symbols constellation
switch bitsPerSubCarrier
    case 2  % QPSK
        refConst = qammod((0:3).', 4, 'UnitAveragePower', true);
    case 4  % 16QAM
        refConst = qammod((0:15).', 16,'UnitAveragePower', true);
    case 6  % 64QAM
        refConst = qammod((0:63).', 64,'UnitAveragePower', true);
    case 8  % 256QAM
        refConst = qammod((0:255).', 256,'UnitAveragePower', true);
end
constDiagRx = comm.ConstellationDiagram( ...
    'ShowReferenceConstellation', true, ...
    'ReferenceConstellation', refConst, ...
    'Position', figposition([20 15 25 30]), ...
    'MeasurementInterval', length(dataRxSymbols), ...
    'Title', 'F-OFDM Demodulated Symbols', ...
    'Name', 'F-OFDM Reception', ...
    'XLimits', [-1.5 1.5], 'YLimits', [-1.5 1.5]);
constDiagRx(dataRxSymbols);
 
% Channel equalization is not necessary here as no channel is modeled
 
% Demapping and BER computation
qamDemod = comm.RectangularQAMDemodulator('ModulationOrder', ...
    2^bitsPerSubCarrier, 'BitOutput', true, ...
    'NormalizationMethod', 'Average power');
BER = comm.ErrorRate;
 
% Perform hard decision and measure errors
rxBits = qamDemod(dataRxSymbols);
ber = BER(bitsIn, rxBits);
 
disp(['F-OFDM Reception, BER = ' num2str(ber(1)) ' at SNR = ' ...
    num2str(snrdB) ' dB']);
 
% Restore RNG state
rng(s);
```

### :chart: 4ASK Fig

![Fig.1  Impulse Response.][1]
<center><font size=2>Fig.1  Impulse Response.</font></center>

![Fig.2  F-OFDM normalized frequency.][2]
<center><font size=2>Fig.2  F-OFDM normalized frequency.</font></center>

![Fig.2  OFDM normalized frequency.][3]
<center><font size=2>Fig.3  OFDM normalized frequency.</font></center>

![Fig.4  F-OFDM Reception.][4]
<center><font size=2>Fig.4  F-OFDM Reception.</font></center>

### :bell: Reference

> 1. Abdoli J., Jia M. and Ma J., "Filtered OFDM: A New Waveform for Future Wireless Systems," 2015 IEEE® 16th International Workshop on Signal Processing Advances in Wireless Communications (SPAWC), Stockholm, 2015, pp. 66-70.
> 2. R1-162152. "OFDM based flexible waveform for 5G." 3GPP TSG RAN WG1 meeting 84bis. Huawei; HiSilicon. April 2016.
> 3. R1-165425. "F-OFDM scheme and filter design." 3GPP TSG RAN WG1 meeting 85. Huawei; HiSilicon. May 2016.

<!-- markdownlint-disable-file MD025 MD033 -->
[1]: https://raw.githubusercontent.com/boom1999/boom1999.github.io/refs/heads/hexo_backup/images/F-OFDM/Impluse_Response.bmp
[2]: https://raw.githubusercontent.com/boom1999/boom1999.github.io/refs/heads/hexo_backup/images/F-OFDM/F-OFDM_normalized_frequency.png
[3]: https://raw.githubusercontent.com/boom1999/boom1999.github.io/refs/heads/hexo_backup/images/F-OFDM/OFDM_normalized_frequency.png
[4]: https://raw.githubusercontent.com/boom1999/boom1999.github.io/refs/heads/hexo_backup/images/F-OFDM/F-OFDM_Reception.png
