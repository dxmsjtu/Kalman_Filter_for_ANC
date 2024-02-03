
# Code Explanation

The section provides a concise introduction to the $KF.mat$ file, which implements the Kalman filter method for a single-channel active noise control (ANC) application. Furthermore, the FxLMS algorithm is conducted as a comparative analysis. The Kalman filter technique employs the modified feed-forward active noise control (ANC) structure, whereas the FxLMS algorithm uses the conventional feed-forward ANC structure.    

## Contents

- [Cleaning the memory and workspace](#cleaning-the-memory-and-workspace)
- [Loading the primary and secondary path](#loading-the-primary-and-secondary-path)
- [Simulation system configuration](#simulation-system-configuration)
- [Creating the disturbance and filtered reference](#creating-the-disturbance-and-filtered-reference)
- [Dynamic noise cancellation by the single channel FxLMS algorithm](#dynamic-noise-cancellation-by-the-single-channel-fxlms-algorithm)
- [Dynamic noise cancellation by the Kalman filter approach](#dynamic-noise-cancellation-by-the-kalman-filter-approach)

## Cleaning the memory and workspace

This segment of code is utilized to clean the memory and workspace of the MATLAB software. 

```matlab
close all ;
clear     ;
clc       ;
```

## Loading the primary and secondary path

This part of the code loads the primary path and secondary path from the Mat files: $PriPath\_3200.mat$ and $SecPath\_200\_6000.mat$. All these paths are synthesized from the band-pass filters, whose impulse responses are illustrated in Figure 1.

```matlab
load('PriPath_3200.mat');
load('SecPath_200_6000.mat') ;
figure ;
subplot(2,1,1)
plot(PriPath);
title('Primary Path');
grid on      ;
subplot(2,1,2);
plot(SecPath);
title('Secondary Path');
xlabel('Taps');
grid on      ;
```

![Primary and Secondary Path](https://github.com/ShiDongyuan/Kalman_Filter_for_ANC/blob/512657bbbed61a13bc0f225c30ee1750e5eed609/Images/KF_01.png)  
Figure 1: The impulse response of the primary path and the secondary path.

## Simulation system configuration

The sampling rate of the active noise control (ANC) system is set to $16000$ Hz, and the simulation duration is $0.25$ second. To simulate the dynamic noise, the primary noise in this ANC system is a chirp signal, whose frequency gradually varies from $20$ Hz to $1600$ Hz, as shown in Figure 2.

```matlab
fs = 16000     ; % sampling rate 16 kHz.
T  = 0.25      ; % Simulation duration (seconds).
t  = 0:1/fs:T  ;% Time variable.
N  = length(t) ;
fw = 500       ;
fe = 300       ;
y = chirp(t,20,T,1600);
figure   ;
plot(t,y);
title('Reference signal x(n)');
xlabel('Time (seconds)') ;
ylabel('Magnitude')      ;
axis([-inf inf -1.05 1.05]);
grid on ;
```

![Reference signal x(n)](KF_02.png)  
Figure 2: The waveform of the reference signal that is a chirp signal ranging from $20$ to $1600$ Hz.

## Creating the disturbance and filtered reference

The disturbance and filtered reference used in the ANC system are created by passing the chirp signal through the loaded primary and secondary paths.

```matlab
%X  = 0.4*sin(2*pi*fw*t)+0.3*sin(2*pi*fe*t);
X = y;
%plot(X(end-100:end))
D  = filter(PriPath,1,X);
Rf = filter(SecPath,1,X);
%plot(D(end-100:end))
```

## Dynamic noise cancellation by the single channel FxLMS algorithm

In this part, the single-channel FxLMS algorithm is used to reduce the chirp disturbance. The length of the control filter in the FxLMS algorithm has $80$ taps, and the step size is set to $0.0005$. Figure 3 shows the error signal picked up by the error sensor in the ANC system. This figure shows that the FxLMS algorithm can not fully attenuate this dynamic noise during the $0.25$ second. 

```matlab
L   = 80    ;
muW = 0.0005;
noiseController = dsp.FilteredXLMSFilter('Length',L,'StepSize',muW, ...
    'SecondaryPathCoefficients',SecPath);
[y,e] = noiseController(X,D);
figure;
plot(t,e) ;
title('FxLMS algorithm')   ;
ylabel('Error signal e(n)');
xlabel('Time (seconds)')   ;
grid on ;
```

![FxLMS algorithm](KF_03.png)   
Figure 3. The error signal of the single-channel ANC system based on the FxLMS algorithm.

## Dynamic noise cancellation by the Kalman filter approach

The Kalman filter is employed in the signal-channel ANC system to track the fluctuation of the chirp disturbance. The variance of the observed noise is initially set to $0.005$. Figure 4 shows the error signal of the Kalman filter algorithm. Additionally, Figure 5 illustrates the variation of the coefficients $w_5(n)$ and $w_{60}(n)$ as time progresses. The outcome illustrates that the Kalman filter effectively mitigates the chirp disturbances. The Kalman filter approach has markedly superior convergence behavior compared to the FxLMS method, as illustrated in Figure 6. 

```matlab
q  = 0.005;
P  = eye(L);
W  = zeros(L,1);
Xd = zeros(L,1);
ek = zeros(N,1);
w5 = zeros(N,1);
w60 = zeros(N,1);

%-----------Kalman Filer---------
for ii =1:N
    Xd     =[Rf(ii);Xd(1:end-1)];
    yt     = Xd'*W ;
    ek(ii) = D(ii)-yt ;
    K      = P*Xd/(Xd'*P*Xd + q);
    W      = W +K*ek(ii)        ;
    P      =(eye(L)-K*Xd')*P    ;
    %---------------------------
    w5(ii)  = W(5);
    w60(ii) = W(60);
    %---------------------------
end
%-------------------------------
figure;
plot(t,ek);
title('Kalman algorithm')   ;
ylabel('Error signal e(n)');
xlabel('Time (seconds)')
grid on ;
figure
plot(t,w5,t,w60);
title('Control Filter Weights');
xlabel('Time (seconds)');
legend('w_5','w_{60}');
grid on ;
figure;
plot(t,e,t,ek);
title('FxLMS vs Kalman')   ;
ylabel('Error signal e(n)');
xlabel('Time (seconds)')   ;
legend('FxLMS algorithm','KF algorithm');
grid on ;
```

![Kalman algorithm](KF_04.png)  
Figure 4: The error signal of the single-channel ANC system based on the Kalman filter.  

![Control Filter Weights](KF_05.png)  
Figure 5: The time history of the coefficients $w_5(n)$ and $w_60(n)$ in the control filter.  

![FxLMS vs Kalman](KF_06.png)  
Figure 6：Comparison of the error signals in the FxLMS algorithm and the Kalman filter.  