---
layout: page
title: modeling a zombie outbreak
description:
img: assets/img/zombies.png 
importance: 2
category: applied
---

## Introduction

What would you do in the doomsday scenario? Well I know what I would do, sit down and mathematically model the outbreak (probably not the best course of action I reckon, but oh well ...). For this project, I extend the work of [Munz et al. (2009)](https://loe.org/images/content/091023/Zombie%20Publication.pdf). I simulate how changing certain parameters affect system stability and the outbreak.

<a href="https://github.com/posmikdc/zombie/blob/main/MATH3006%20Final%20Project.pdf" target="_blank">
    <img src="https://blogs.mathworks.com/images/pick/Sean/mainZombie/mainZombie_02.png" 
    alt="IMAGE ALT TEXT HERE" width="240" height="180" border="10" />
</a>

## Set up ODEs

```{m}
%Math Modeling, Zombie Outbreak
clear all; close all; clc;

%% Part 1: SZR Model
clear all; close all; clc;

syms S %Population of susceptible people
syms Z %Population of zombies
syms R %Population of removed people and zombies

syms Sstar
syms Zstar
syms Rstar

syms delta %Non zombie related deaths
syms zeta %Resurrected humans that turn into zombies
syms beta %Zombie encounter transmission parameter
syms m %Birth rate (constant)
syms alpha %Defeated zombie parameter
syms N %Total population

%Numerical values, "N" denotes numerical variables instead of symbolic variable
alphaN = 0.005; %Test 0.002, 0.008
betaN = 0.0095; %Test 0.0065, 0.0125
deltaN = 0.0001; 
zetaN = 0.0001; 
Npop = 500;

%Set up ODE
T = 10; %Total number of days
dt = 1; %Change in unit time (one day)
n = T/dt;
t = zeros(1,n+1);
s = zeros(1,n+1);
z = zeros(1,n+1);
r = zeros(1,n+1);
t = 0:dt:T;

%Assume: Short period of time
m = 0;
delta = 0;

%ODE
Sprime = m - (beta*S*Z) - (delta*S);
Zprime = (beta*S*Z) + (zeta*R) - (alpha*S*Z);
Rprime = (delta*S) + (alpha*S*Z) - (zeta*R);

J = jacobian([Sprime, Zprime, Rprime], [S,Z,R]); %Initial jacobian matrix
J1 = subs(J, [S,Z,R] , [N,0,0]); %Jacobian matrix of first steady state
J2 = subs(J, [S,Z,R] , [0,Zstar,0]); %Jacobian matrix of second steady state

%Identity matrix
syms lambda
I = [lambda,0,0; 0,lambda,0; 0,0,lambda];
J1 = J1-I;
J2 = J2-I;

%Determinant of (J-lambdaI)
dJ1 = det(J1);
dJ2 = det(J2);

dJ1 = vpa(subs(dJ1, [alpha, beta, zeta, N], [alphaN, betaN, zetaN, Npop]),3);
dJ2 = vpa(subs(dJ2, [alpha, beta, zeta, N, Zstar], [alphaN, betaN, zetaN, Npop, 1]),3);

%Solve for roots of polynomial equation
L1 = vpa(solve(dJ1),3);
L2 = vpa(solve(dJ2),3);

%Switch back to numerical values to solve ODE and graph
s(1) = Npop; 
z(1) = 1;
r(1) = 0;

for x = 1:n
    s(x+1) = s(x) + dt*(-betaN*s(x)*z(x));
    z(x+1) = z(x) + dt*(betaN*s(x)*z(x) - alphaN*s(x)*z(x) + zetaN*r(x));
    r(x+1) = r(x) + dt*(alphaN*s(x)*z(x) + deltaN*s(x) - zetaN*r(x));
 
   %Assume: S, Z, and R > 0 and S, Z, and R < N
   if s(x+1) < 0
        s(x+1) = 0;
    end
    if s(x+1) > Npop
        s(x+1) = Npop;
    end
    
    if z(x+1) < 0
        z(x+1) = 0;
    end
    if z(x+1) > Npop
        z(x+1) = Npop;
    end
    
    if r(x+1) < 0
        r(x+1) = 0;
    end
    if r(x+1) > Npop
        r(x+1) = Npop;
    end
end

hold on
plot(t,s,'b');
plot(t,z,'r');
plot(t,r,'m');
xlabel('Days');
ylabel('Population');
legend('Suscepties','Zombies','Removed')
title('Change in SZR Popluation');
axis([0, T, 0, Npop])


%% Part 2: SIZR Model
clear all; close all; clc;

syms S %Population of susceptible people
syms I %Population of infected people
syms Z %Population of zombies
syms R %Population of removed people and zombies

syms Sstar
syms Istar
syms Zstar
syms Rstar

syms delta %Non zombie related deaths
syms zeta %Resurrected humans that turn into zombies
syms beta %Zombie encounter transmission parameter
syms m %Birth rate (constant)
syms alpha %Defeated zombie parameter
syms rho
syms N %Total population

%Numerical values
alphaN = 0.001; %0.005
betaN = 0.0095; %0.0095
deltaN = 0.001; %0.0001
zetaN = 0.05; %0.0001
rhoN = 0.15; %Test 0.05, 0.10, 0.15
Npop = 500;

%Set up ODE
T = 30; %Total number of days
dt = 1; %Change in unit time (one day)
n = T/dt;
t = zeros(1,n+1);
s = zeros(1,n+1);
i = zeros(1,n+1);
z = zeros(1,n+1);
r = zeros(1,n+1);
t = 0:dt:T;

%Assume: Short period of time
m = 0;
delta = 0;

%ODE
Sprime = m - (beta*S*Z) - (delta*S);
Iprime = (beta*S*Z) - (rho*I) - (delta*I);
Zprime = (rho*I) + (zeta*R) - (alpha*S*Z);
Rprime = (delta*S) + (delta*I) + (alpha*S*Z) - (zeta*R);

J = jacobian([Sprime, Iprime, Zprime, Rprime], [S,I,Z,R]); %Initial jacobian matrix
J1 = subs(J, [S,I,Z,R] , [N,0,0,0]); %Jacobian matrix of first steady state
J2 = subs(J, [S,I,Z,R] , [0,0,Zstar,0]); %Jacobian matrix of second steady state

%Identity matrix
syms lambda
I = [lambda,0,0,0; 0,lambda,0,0; 0,0,lambda,0; 0,0,0,lambda];
J1 = J1-I;
J2 = J2-I;

%Determinant of (J-lambdaI)
dJ1 = det(J1);
dJ2 = det(J2);

dJ1 = vpa(subs(dJ1, [alpha, beta, zeta, rho, N], [alphaN, betaN, zetaN, rhoN, Npop]),3);
dJ2 = vpa(subs(dJ2, [alpha, beta, zeta, N, rho, Zstar], [alphaN, betaN, zetaN, rhoN, Npop, 1]),3);

%Solve for roots of polynomial equation
L1 = vpa(solve(dJ1),3);
L2 = vpa(solve(dJ2),3);

%Switch back to numerical values to solve ODE and graph
s(1) = Npop; 
i(1) = 0;
z(1) = 1;
r(1) = 0;

for x = 1:n
    s(x+1) = s(x) + dt*(-betaN*s(x)*z(x) - delta*s(x));
    i(x+1) = i(x) + dt*(betaN*s(x)*z(x) - rhoN*i(x));
    z(x+1) = z(x) + dt*(rhoN*i(x) + zetaN*r(x) - alphaN*s(x)*z(x));
    r(x+1) = r(x) + dt*(alphaN*s(x)*z(x) - zetaN*r(x));
 
   %Assume: S, I, Z, and R > 0 and S, I, Z, and R < N
   if s(x+1) < 0
        s(x+1) = 0;
    end
    if s(x+1) > Npop
        s(x+1) = Npop;
    end
    
     if i(x+1) < 0
        i(x+1) = 0;
    end
    if i(x+1) > Npop
        i(x+1) = Npop;
    end
    
    if z(x+1) < 0
        z(x+1) = 0;
    end
    if z(x+1) > Npop
        z(x+1) = Npop;
    end
    
    if r(x+1) < 0
        r(x+1) = 0;
    end
    if r(x+1) > Npop
        r(x+1) = Npop;
    end
end

hold on
plot(t,s,'b');
plot(t,i,'g');
plot(t,z,'r');
plot(t,r,'m');
xlabel('Days');
ylabel('Population');
legend('Suscepties','Infected','Zombies','Removed')
title('Change in SIZR Popluation');
axis([0, T, 0, Npop])

%% Part 3: SIZR Model with treatment
clear all; close all; clc;

syms S %Population of susceptible people
syms I %Population of infected people
syms Z %Population of zombies
syms R %Population of removed people and zombies

syms Sstar
syms Istar
syms Zstar
syms Rstar

syms delta %Non zombie related deaths
syms zeta %Resurrected humans that turn into zombies
syms beta %Zombie encounter transmission parameter
syms m %Birth rate (constant)
syms alpha %Defeated zombie parameter
syms rho %Infectious period parameter
syms N %Total population
syms c %Cure parameter

alphaN = 0.001; %0.005
betaN = 0.0095; %0.0095
deltaN = 0.001; %0.0001
zetaN = 0.05; %0.0001
rhoN = 0.05;
Npop = 500;
cN = 0.43; %Test 0.1, 0.3, 0,5

%Set up ODE
T = 60; %Total number of days
dt = 1; %Change in unit time (one day)
n = T/dt;
t = zeros(1,n+1);
s = zeros(1,n+1);
i = zeros(1,n+1);
z = zeros(1,n+1);
r = zeros(1,n+1);
t = 0:dt:T;

%Assume: Short period of time
m = 0;
delta = 0;

%ODE
Sprime = m - (beta*S*Z) - (delta*S) + (c*Z);
Iprime = (beta*S*Z) - (rho*I) - (delta*I);
Zprime = (rho*I) + (zeta*R) - (alpha*S*Z) - (c*Z);
Rprime = (delta*S) + (delta*I) + (alpha*S*Z) - (zeta*R);

J = jacobian([Sprime, Iprime, Zprime, Rprime], [S,I,Z,R]); %Initial jacobian matrix
J1 = subs(J, [S,I,Z,R] , [N,0,0,0]); %Jacobian matrix of first steady state
J2 = subs(J, [S,I,Z,R] , [0,0,Zstar,0]); %Jacobian matrix of second steady state

%Identity matrix
syms lambda
I = [lambda,0,0,0; 0,lambda,0,0; 0,0,lambda,0; 0,0,0,lambda];
J1 = J1-I;
J2 = J2-I;

%Determinant of (J-lambdaI)
dJ1 = det(J1);
dJ2 = det(J2);

dJ1 = vpa(subs(dJ1, [alpha, beta, zeta, rho, c, N], [alphaN, betaN, zetaN, rhoN, cN, Npop]),3);
dJ2 = vpa(subs(dJ2, [alpha, beta, zeta, N, rho, c, Zstar], [alphaN, betaN, zetaN, rhoN, cN, Npop, 1]),3);

%Solve for roots of polynomial equation
L1 = vpa(solve(dJ1),3);
L2 = vpa(solve(dJ2),3);

%Switch back to numerical values to solve ODE and graph
s(1) = Npop;
i(1) = 0;
z(1) = 1;
r(1) = 0;

for x = 1:n
    s(x+1) = s(x) + dt*(-betaN*s(x)*z(x) + cN*z(x));
    i(x+1) = i(x) + dt*(betaN*s(x)*z(x) - rhoN*i(x));
    z(x+1) = z(x) + dt*(rhoN*i(x) + zetaN*r(x) - alphaN*s(x)*z(x) - cN*z(x));
    r(x+1) = r(x) + dt*(alphaN*s(x)*z(x) - zetaN*r(x));
 
   %Assume: S, I, Z, and R > 0 and S, I, Z, and R < N
   if s(x+1) < 0
        s(x+1) = 0;
    end
    if s(x+1) > Npop
        s(x+1) = Npop;
    end
    
     if i(x+1) < 0
        i(x+1) = 0;
    end
    if i(x+1) > Npop
        i(x+1) = Npop;
    end
    
    if z(x+1) < 0
        z(x+1) = 0;
    end
    if z(x+1) > Npop
        z(x+1) = Npop;
    end
    
    if r(x+1) < 0
        r(x+1) = 0;
    end
    if r(x+1) > Npop
        r(x+1) = Npop;
    end
end

hold on
plot(t,s,'b');
plot(t,i,'g');
plot(t,z,'r');
plot(t,r,'m');
xlabel('Days');
ylabel('Population');
legend('Suscepties','Infected','Zombies','Removed')
title('Change in SIZR Popluation');
axis([0, T, 0, Npop])

%% Part 4: SIZRQ Model
clear all; close all; clc;

alphaN = 0.001; %0.005
betaN = 0.0095; %0.0095
deltaN = 0.001; %0.0001
zetaN = 0.05; %0.0001
rhoN = 0.05; 
Npop = 500;
sigmaN = 0.03; %Zombie population entering quarantine
kN = 0.03; %Infected popluation entering quarantine
gammaN = 0.05; %Quarantine popluation being removed


%Set up ODE
T = 60; %Total number of days
dt = 1; %Change in unit time (one day)
n = T/dt;
t = zeros(1,n+1);
s = zeros(1,n+1);
i = zeros(1,n+1);
z = zeros(1,n+1);
r = zeros(1,n+1);
q = zeros(1,n+1);
t = 0:dt:T;

%Switch back to numerical values to solve ODE and graph
s(1) = Npop;
i(1) = 0;
z(1) = 1;
r(1) = 0;
q(1) = 0;

for x = 1:n
    s(x+1) = s(x) + dt*(-betaN*s(x)*z(x) - deltaN*s(x));
    i(x+1) = i(x) + dt*(betaN*s(x)*z(x) - rhoN*i(x) - deltaN*i(x) - kN*i(x));
    z(x+1) = z(x) + dt*(rhoN*i(x) + zetaN*r(x) - alphaN*s(x)*z(x) - sigmaN*z(x));
    r(x+1) = r(x) + dt*(deltaN*s(x) + deltaN*i(x) + alphaN*s(x)*z(x) - zetaN*r(x) + gammaN*q(x));
    q(x+1) = q(x) + dt*(kN*i(x) + sigmaN*z(x) - gammaN*q(x));
    
   %Assume: S, I, Z, R, Q > 0 and S, I, Z, R, Q < N
   if s(x+1) < 0
        s(x+1) = 0;
    end
    if s(x+1) > Npop
        s(x+1) = Npop;
    end
    
     if i(x+1) < 0
        i(x+1) = 0;
    end
    if i(x+1) > Npop
        i(x+1) = Npop;
    end
    
    if z(x+1) < 0
        z(x+1) = 0;
    end
    if z(x+1) > Npop
        z(x+1) = Npop;
    end
    
    if r(x+1) < 0
        r(x+1) = 0;
    end
    if r(x+1) > Npop
        r(x+1) = Npop;
    end
    
     if q(x+1) < 0
        q(x+1) = 0;
    end
    if q(x+1) > Npop
        q(x+1) = Npop;
    end
end

hold on
plot(t,s,'b');
plot(t,i,'g');
plot(t,z,'r');
plot(t,r,'m');
plot(t,q,'k');
xlabel('Days');
ylabel('Population');
legend('Suscepties','Infected','Zombies','Removed','Quarantine')
title('Change in SIZRQ Popluation');
axis([0, T, 0, Npop])
```
