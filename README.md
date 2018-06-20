# pendulum-data

close all;

g       = 9.81;                 % Gravatational Constant            [m/s^2]
I       = 248729/(1000^3);      % Moment of Inertia about Piviot    [kg-m^2]
l_cm    = 20.8/1000 ;           % Piviot to CM                      [m]
b       = .00025;               % Viscus Damping                    [N-m-s/rad]
m       = 114/1000;             % Mass                              [kg]
sigma   = 0;                    % Coloumb Friction Coeffieicnet     [N-m]

z = iddata(y,[],.015148008,'Name','Pendulum');
z.OutputName = 'Pendulum position';
z.OutputUnit = 'rad';
z.TimeUnit = 's';

figure('Name',[z.Name ': output data']);
plot(z);

FileName        = 'pendulum';
Order           = [1 0 2];
Parameters      = [g; I; l_cm; b; m; sigma];
InitialStates   = [y(1); 0];
Ts              = 0;

nlgr = idnlgrey(FileName,Order,Parameters,InitialStates,Ts);
nlgr.OutputName = 'Pendulum Position';
nlgr.OutputUnit = 'rad';
nlgr.TimeUnit = 's';
nlgr = setinit(nlgr, 'Name', {'Pendulum Position' 'Pendulum Velocity'});
nlgr = setinit(nlgr, 'Unit', {'rad' 'rad/s'});
nlgr = setpar(nlgr, 'Name', {'Gravity Constant' 'Moment of Inertia' 'Radius' 'Viscus Damping' 'Mass' 'Coluomb Friction'});
nlgr = setpar(nlgr, 'Unit', {'m/s^2' 'kg-m^2' 'm' 'Nms/rad' 'kg' 'Nm'});
nlgr = setpar(nlgr, 'Minimum', {eps(0) eps(0) eps(0) eps(0) eps(0) 0});

nlgrrk45.SimulationOptions.Solver = 'ode45';     % Runge-Kutta 45.
compare(z, nlgrrk45, 1, ...
   compareOptions('InitialCondition', 'model'));


nlgrrk45 = setpar(nlgrrk45, 'Fixed', {true true true false true true});

opt = nlgreyestOptions('Display', 'on');
trk45 = clock;
nlgrrk45 = nlgreyest(z, nlgrrk45, opt);   % Perform parameter estimation.
trk45 = etime(clock, trk45);

compare(z,nlgrrk45, 1, ...
   compareOptions('InitialCondition', 'model'));
