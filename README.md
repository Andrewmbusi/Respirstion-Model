%% DYNAMIC ALVEOLAR GAS EXCHANGE MODEL
clc;clear;close all;
%% SECTION 1: PHYSIOLOGICAL CONSTANTS
%% ------------------------------------------------------------
Patm = 760;              % Atmospheric pressure (mmHg)
PH2O = 47;               % Water vapor pressure (mmHg)
Pdry = Patm - PH2O;      % Dry gas pressure
FIO2 = 0.21;             % Fraction of inspired oxygen (dimensionless).
FICO2 = 0.0;             % Fraction of inspired carbon dioxide (dimensionless).
PIO2 = FIO2 * Pdry;      % Inspired oxygen partial pressure (mmHg).
PICO2 = 0;               % Inspired carbon dioxide partial pressure (mmHg).
PvO2 = 40;               % Venous oxygen pressure (mmHg)
PvCO2 = 46;              % Venous CO2 pressure (mmHg)
VA = 3.0;                % Alveolar volume (L)
Vc = 0.1;                % Capillary volume (L)
Qdot = 5.0;              % Blood perfusion rate (L/min)
DL_O2 = 0.021;           % Diffusion capacity O2 (L/min/mmHg)
DL_CO2 = 0.45;           % CO2 diffuses ~20x faster
%% SECTION 2: VENTILATION CASES
%% ------------------------------------------------------------
ventilation_cases = [4, 2, 8];   % Normal, Hypo, Hyper
case_names = {'Normal','Hypoventilation','Hyperventilation'};
%% SECTION 3: TIME SETTINGS
%% ------------------------------------------------------------
tspan = [0 15];
%% SECTION 4: INITIAL CONDITIONS
y0 = [100, 40, 40, 46];
%% SECTION 5: STORAGE VARIABLES
%% ------------------------------------------------------------
rez = cell(3,1);
%% SECTION 6: MAIN SIMULATION LOOP
%% ------------------------------------------------------------
for i=1:3
VA_doti=ventilation_cases(i);
%% ODE SYSTEM
f=@(t,y)[
(VA_doti/VA)*(PIO2-y(1))-(DL_O2/VA)*(y(1)-y(2));   % Change in alveolar O2 = ventilation input − diffusion into blood
(DL_O2/Vc)*(y(1)-y(2))-(Qdot/Vc)*(y(2)-PvO2);     % Change in capillary O2 = diffusion from alveoli − blood flow to tissues
(VA_doti/VA)*(PICO2-y(3))-(DL_CO2/VA)*(y(3)-y(4)); % Change in alveolar CO2 = ventilation removal − diffusion from blood
(DL_CO2/Vc)*(y(3)-y(4))-(Qdot/Vc)*(y(4)-PvCO2)    % Change in capillary CO2 = diffusion into alveoli − blood flow from tissues
];
%% SOLVE ODE
[t,y]=ode45(f,tspan,y0);
rez{i}.t=t;
rez{i}.y=y;
rez{i}.VA_doti=VA_doti;
end
%% SECTION 7: PLOTTING OXYGEN
%% ------------------------------------------------------------
figure;
subplot(2,1,1)
hold on
grid on
for i=1:3
plot(rez{i}.t,rez{i}.y(:,1),'LineWidth',3)
end
xlabel('Time (min)')
ylabel('Alveolar PO2 (mmHg)')
title('Alveolar Oxygen Comparison')
legend(case_names)
%% SECTION 8: PLOTTING CO2
%% ------------------------------------------------------------
subplot(2,1,2)
hold on
grid on
for i=1:3
plot(rez{i}.t,rez{i}.y(:,3),'LineWidth',3)
end
xlabel('Time (min)')
ylabel('Alveolar PCO2 (mmHg)')
title('Alveolar CO2 Comparison')
legend(case_names)
%% SECTION 9: CAPILLARY PLOTS
%% ------------------------------------------------------------
figure;
subplot(2,1,1)
hold on
grid on
for i=1:3
plot(rez{i}.t,rez{i}.y(:,2),'LineWidth',3)
end
xlabel('Time (min)')
ylabel('Capillary PO2 (mmHg)')
title('Capillary Oxygen')
legend(case_names)
subplot(2,1,2)
hold on
grid on
for i=1:3
plot(rez{i}.t,rez{i}.y(:,4),'LineWidth',3)
end
xlabel('Time (min)')
ylabel('Capillary PCO2 (mmHg)')
title('Capillary CO2')
legend(case_names)
%% SECTION 10: STEADY STATE OUTPUT
%% ------------------------------------------------------------
fprintf('\n------------------------------------\n')
fprintf('STEADY STATE RESULTS\n')
fprintf('------------------------------------\n')
for i=1:3
y_end=rez{i}.y(end,:);
fprintf('\n%s\n',case_names{i})
fprintf('Ventilation = %.2f L/min\n',rez{i}.VA_doti)
fprintf('Alveolar PO2 = %.2f mmHg\n',y_end(1))
fprintf('Capillary PO2 = %.2f mmHg\n',y_end(2))
fprintf('Alveolar PCO2 = %.2f mmHg\n',y_end(3))
fprintf('Capillary PCO2 = %.2f mmHg\n',y_end(4))
end
fprintf('\nSimulation complete.\n')

