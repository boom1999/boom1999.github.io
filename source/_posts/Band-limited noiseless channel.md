---
title: Band-limited noiseless channel
date: 2021-05-25
tags:
   - modulate
categories: Matlab
copyright: true
---

:pushpin:

### Consider a band-limited noiseless channel. The channel response in frequency domain is H(f)=1, |f|≤3Hz, otherwise, H(f)=0 ###

:umbrella::clock1130::sleeping:

<!--more-->

---

### :memo: Code ###

- _main.m_

``` matlab
%% Simulate the band-limited channel.

clear;
clc;

Ts = 1;                 % Symbol interval.
Tb = Ts/3;              % Bit interval.
fs = 1000;              % Sampling rate.
dt = 1/fs;              % Sampling interval.
N = 10;                 % Number of symbols.
t = -N/2 : dt : N/2;    % Sequence transmission time. 
m = [0,1,2,3,4,5,6,7];  % Equally spaced.
Am = -3.5+m;            % The amplitude of the m-th waveform.

%% Task(1) Randomly generate a sequence of ten 8-PAM signals.
for i=1:length(t)
    if(fix((i-1)/fs)==((i-1)/fs))
        temp=randi(7);
        Sm(i)=Am(temp);
    else
        Sm(i)=Sm(i-1);
    end
end
[f,Sf] = T2F(t,Sm);      % TF

figure(1)
plot(t,Sm);
xlabel('t');ylabel('Amplitude '); 
title('8-PAM waveform');

figure(2)
plot(f,abs(Sf));
xlabel('f/Ts' );ylabel('');
title('8-PAM spectrum');

%% Task(2) Consider a band-limited noiseless channel.
% Generate a band-limited noiseless channel H(f)=1, |f|≤3Hz.
for k = 1 : length(f)
    if abs(f(k))>3
        Hf1(k)=0;
    else
        Hf1(k)=Ts;
    end
end
for k = 1 : length(f)
    if abs(f(k))>300
        Hf2(k)=0;
    else
        Hf2(k)=Ts;
    end
end

% Plot the band-limited noiseless channel.
figure(3)
plot(f,Hf1,f,Hf2);
grid on;
xlabel ('f/Ts'); ylabel('Noiseless channel spectrum');
axis([ - 400 400 0  1.2]);
legend('cut-off f=3','cut-off f=300');


% Passing through this channel.
value_t=t(1);
Xf1 = Sf.*Hf1;
Xf1 = Xf1.*exp(1i*2*pi*f*value_t);
[t,Xt1] = F2T(f,Xf1);      % FT

Xf2 = Sf.*Hf2;
Xf2 = Xf2.*exp(1i*2*pi*f*value_t);
[t,Xt2] = F2T(f,Xf2);      % FT

figure(4)
subplot(121)
plot(f,abs(Xf1));
xlabel('f/Ts');ylabel(''); 
axis([ -5 5 0 6]);
title('8-PAM waveform passomg noiseless channel spectrum');
legend('cut-off f=3');
subplot(122)
plot(f,abs(Xf2));
xlabel('f/Ts');ylabel(''); 
axis([ -50 50 0 6]);
title('8-PAM waveform passomg noiseless channel spectrum');
legend('cut-off f=300');

figure(5)
subplot(121)
plot(t,real(Xt1));
axis([ 0 N -4 4]);
xlabel('t');ylabel('Amplitude'); 
title('8-PAM waveform passing noiseless channel waveform');
legend('cut-off f=3');

subplot(122)
plot(t,real(Xt2));
axis([ 0 N -4 4]);
xlabel('t');ylabel('Amplitude'); 
title('8-PAM waveform passing noiseless channel waveform');
legend('cut-off f=300');
```

- _T2F.m_

``` matlab
function [f,sf]=T2F(t,st)
%input is time and the signal vectors
%output is freguency and signal spectrum

dt=t(2)-t(1);
T=t(end)-t(1)+dt;
df=1/T; %smapling rate
N=length(st);

f=-N/2*df:df:(N/2-1)*df; %频域抽样点
sf=fft(st);
sf=T/N*fftshift(sf).*exp(-j*2*pi*f*t(1));   %补偿时间移位
```

- _F2T.m_

``` matlab
%% Inverse Fourier Transform
function [t,st] = F2T(f,sf)
% output is time ande signal sequence
% input is frequency and signal spectrum

df = f(2) - f(1);
Fmx = f(end) - f(1) + df;
dt = 1/Fmx;                 % sampling rate
N = length(sf);
T = N*dt;

t = 0:dt:T-dt;              % time sequence
sff = ifftshift(sf);
st = Fmx*ifft(sff);
```

### :chart: Fig ###

![Fig.1  8-PAM waveform.][1]
<center><font size=2>Fig.1  8-PAM waveform.</font></center>
<br><br><br>

![Fig.2  8-PAM spectrum.][2]
<center><font size=2>Fig.2  8-PAM spectrum.</font></center>
<br><br><br>

![Fig.3  Noiseless channel spectrum.][3]
<center><font size=2>Fig.3  Noiseless channel spectrum.</font></center>
<br><br><br>

![Fig.4  8-PAM waveform passing noiseless channel spectrum.][4]
<center><font size=2>Fig.4  8-PAM waveform passing noiseless channel spectrum.</font></center>
<br><br><br>

![Fig.5  8-PAM waveform passing noiseless channel waveform.][5]
<center><font size=2>Fig.5  8-PAM waveform passing noiseless channel waveform.</font></center>
<br><br><br>

### :grey_question: Discussion ###

1. The response of the noise-free band-limited channel is lower than the rectangular window of H(f)=1 at the cut-off frequency. After the 8-PAM signal passes through the band-limited channel, the high-frequency information is filtered out, and the restored information waveform becomes smooth; if continue to increase the cut-off frequency, the restored signal waveform will contain more original information, and the same bit error rate will decrease accordingly.
2. The result of time domain convolution is the same as the result of inverse Fourier transform after frequency multiplication.
3. Under a band-limited noiseless channel, the larger the passband, the smoother the signal.
<!-- markdownlint-disable-file MD025 MD033 -->
[1]: https://www.lingzhicheng.cn/usr/file/picture/Matlab/Band_limited_channel/8-PAM_waveform.png
[2]: https://www.lingzhicheng.cn/usr/file/picture/Matlab/Band_limited_channel/8-PAM_spectrum.png
[3]: https://www.lingzhicheng.cn/usr/file/picture/Matlab/Band_limited_channel/Noiseless_channel_spectrum.png
[4]: https://www.lingzhicheng.cn/usr/file/picture/Matlab/Band_limited_channel/8-PAM_waveform_passing_noiseless_channel_spectrum.png
[5]: https://www.lingzhicheng.cn/usr/file/picture/Matlab/Band_limited_channel/8-PAM_waveform_passing_noiseless_channel_waveform.png
