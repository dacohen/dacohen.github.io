---
layout: post
title:  "Tracking the ISS: Part 1 - Positioning"
date:   2019-07-28 11:35:53 -0400
categories: science iss
background: '/images/iss1.jpg'
---

<link rel="stylesheet" href="/css/monokai.css">
{% include mathjax.html %}

If you're just joining us now, and need a quick refresher on orbital mechanics, check out the preceeding post [here]({% post_url 2019-07-28-tracking-the-iss-part-0-prerequisites %}). 

I've posted the full code described below. It's available in a GitHub repository here:
<https://github.com/dacohen/orbitalprediction/tree/master/part1>

# Using TLEs (Two-Line Elements)
TLEs are a generic format for distributing orbital parameters. In this case, NASA generates an updated set of TLEs for the ISS each day. We can go to their [website](https://spaceflight.nasa.gov/realdata/sightings/SSapplications/Post/JavaSSOP/orbit/ISS/SVPOST.html) and grab a TLE for the ISS. Here's the one I got:

```language-plaintext
ISS
1 25544U 98067A   19209.53234192  .00016717  00000-0  10270-3 0  9029
2 25544  51.6398 156.1486 0006337 192.2040 167.8958 15.50992959 21665
```

Each of these fields has a specific meaning, fully explained [here](https://en.wikipedia.org/wiki/Two-line_element_set#Format). We could take the time to write a parser for the TLE, but that's not very interesting, and there's lots of libraries already available that do just that. Instead, I'll just copy out the interesting values by hand and add the correct units:


<table border="1" cellpadding="10">
	<thead>
		<tr class="header">
			<th>Parameter</th>
			<th>Value</th>
			<th>Unit</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>Epoch Time</td>
			<td>19209.53234192</td>
			<td>yrday.fracday</td>
		</tr>
		<tr>
			<td>RAAN</td>
			<td>156.1486</td>
			<td>deg</td>
		</tr>
		<tr>
			<td>Eccentricity</td>
			<td>0.00006337</td>
			<td></td>
		</tr>
		<tr>
			<td>Arg of perigee</td>
			<td>192.2040</td>
			<td>deg</td>
		</tr>
		<tr>
			<td>Mean anomaly</td>
			<td>167.8958</td>
			<td>deg</td>
		</tr>
		<tr>
			<td>Mean motion</td>
			<td>15.50992959</td>
			<td>\(\frac{rev}{day}\)</td>
		</tr>
		<tr>
			<td>Decay rate</td>
			<td>\(1.67170 \times 10^{-4}\)</td>
			<td>\(\frac{rev}{day^2}\)</td>
		</tr>
	</tbody>
</table>

Note: The TLE elements above are designed to be used with an [SGP](https://en.wikipedia.org/wiki/Simplified_perturbations_models) model. As such, the predictions produced below will have lower accuracy because they don't take into account factors such as the spheriod shape of the earth. If you're planning an ISS rendezvous, don't use the results from the model we're about to create. 


# Helper Methods

The Epoch Time is written as the last two digits of the year (19 for 2019) and then the fractional number of days since the start of that year. In this case, the Epoch happens 209.53234192 fractional days after the beginning of 2019. Let's write a few helper functions to handle this input.

Let's create a folder for this project, and then create a file called *utils.py* for these functions, starting with one to parse the epoch time format from the TLE:

```python
def datetime_from_epoch(epoch):
	(date_part, fractional_part) = epoch.split(".")
	fractional_part = "0.%s" % fractional_part
	date = datetime.datetime.strptime(date_part, "%y%j")
	date += datetime.timedelta(
		seconds=86400*float(fractional_part))

	return date
```

Next, let's write some functions to compute the Julian Date and Siderial time, we'll need these later:
```python
def julian_date(time):
	UT = (time.minute / 60.0) + time.hour
	JD = (367*time.year) - \
		int((7*(time.year+int((time.month+9)/12)))/4) + \
		int((275*time.month)/9) + time.day + 1721013.5 + (UT/24)
	return JD

def greenwich_siderial_time(time):
	GMST = 18.697374558 + 24.06570982441908*(julian_date(time) - 2451545)
	return GMST % 24
```

These formulas come from the USNO (United States Naval Observatory) [page](https://web.archive.org/web/20181113033123/http://aa.usno.navy.mil/faq/docs/JD_Formula.php) of [formulas](https://web.archive.org/web/20190524114447/https://aa.usno.navy.mil/faq/docs/GAST.php).

 Let's create another file called *constants.py*, and put the following physical constants in it:
```python
 # Constants
# Mass of the Earth in kg
M = 5.9722e24

# Gravitational constant
G = 6.67430e-11
```

# Propagating Orbit Parameters

Now for the fun part! Let's create one more file, called *predict.py*, import a few packages, and put in the values we extracted from the TLE:
```python
import datetime
import math
import utils
import constants

# TLE variables for ISS.
epoch = "19209.53234192"
inclination = 51.6398 # degrees
raan = 156.1486 # degrees
eccentricity = 0.0006337
arg_of_perigee = 192.2040 # degrees
mean_anomaly = 167.8958 #degrees
mean_motion = 15.50992959 #rev/day
decay_rate = 1.6717e-4 # rev/day^2
```

Now, lets standardize everything to radians, the unit of champions, and seconds:
```python
# Derived variables
utcEpoch = utils.datetime_from_epoch(epoch)
inclination_rad = math.radians(inclination)
raan_rad = math.radians(raan)
arg_of_perigee_rad = math.radians(arg_of_perigee)
mean_anomaly_rad = math.radians(mean_anomaly)
mean_motion_rps = mean_motion * (2 * math.pi) / 86400
decay_rate_rps2 = decay_rate * (2 * math.pi) / 86400**2
```

Note that when we convert from square days to square seconds, we have to square the conversion factor. That one tripped me up for a few hours.

We also need to find the size of the orbit. We know the period of the circular orbit from the mean motion, and we know that an elliptical orbit has a period equal to that of a circular orbit with a radius equal to its semi-major axis. From basic physics, we know that this semi-major axis is:
\\[a = \sqrt[3]{\frac{GMT}{4\pi^2}}\\]

Basic geometry also gives us that the semi-minor axis is:
\\[b = a\sqrt{1 - e^2}\\]

In python that looks like this:
```python
period_seconds = (2 * math.pi) / mean_motion_rps
semi_major_meters = ((constants.G*constants.M*period_seconds) / (4 * math.pi**2))**(1.0/3.0)
semi_minor_meters = semi_major_meters * math.sqrt(1 - eccentricity**2)
```


Let's also get the time we want to predict the position at. For now, we can choose the current time:
```python
# Inputs
time_to_predict = datetime.datetime.utcnow()

# Let's get started
print("Epoch: %s" % utcEpoch)
print("Predict At: %s" % time_to_predict)
```

Then we need to figure out how many seconds have elapsed since the epoch, which is easy with the datetime library:
```python
# Propagate the mean anomaly
# time since epoch
delta_T = time_to_predict - utcEpoch
delta_T_seconds = delta_T.total_seconds()
```

Now, we can compute the new mean anomaly. Remember, the mean anomaly assumes an imaginary circular orbit, so we can use the mean motion from the TLE, and assume a constant acceleration of:
\\[\frac{dM}{dt} = MeanMotion - DecayRate \times t\\]

To get the current Mean anomaly, we integrate both sides, and get:
\\[M = M_0 + (MeanMotion \times t) - (\frac{1}{2}DecayRate \times t^2)\\]


Now we just have to write that equation in python:
```python
M = (mean_anomaly_rad +
	(mean_motion_rps * delta_T_seconds) -
	(0.5 * decay_rate_rps2 * delta_T_seconds**2)) % (2 * math.pi)
```

I threw in a modulus, so we can make sure the Mean Anomaly is always between 0 and \\(2\pi\\).

Now that we have the Mean Anomaly of our satellite at the prediction time, we just have to solve for the Eccentric Anomaly using Kepler's Equation. Unfortunately, algebra won't help us much. The sine is a transcendental function, so there's no way to solve for E algebraically. But don't worry, there's another way.

## Newton's Method
We can use [Newton's Method](https://en.wikipedia.org/wiki/Newton%27s_method) to find the root of Kepler's equation. This should be quite fast since the ISS has a nearly circular orbit and the Eccentic and Mean anomalies should be almost equal. Since Newton's method finds the zeros of a function, we need to rewrite Kepler's equation so it's equal to zero:
\\[f(E) = E - e \sin(E) - M = 0\\]

Let's also compute the deriviative of \\(f(E)\\):
\\[f'(E) = 1 - e \cos(E)\\]

Now, starting with the Mean Anomaly M, we can iteratively refine our estimate of the Eccentic Anomaly E:
\\[E_{i+1} = E_i - \frac{f(E_i)}{f'(E_i)}\\]

When the fractional part gets suitably small, we're confident we've found the root, and can stop. In python, that looks like this:
```python
# Solve Kepler's equation using Newton's method
E = M
while True:
	delta_E = (E - eccentricity * math.sin(E) - M) / (1 - eccentricity * math.cos(E))
	E -= delta_E
	if math.fabs(delta_E) < 1e-9:
		break
```

When the loop finishes after a few iterations, we'll have a reasonable estimate of E. Given E, we can find the coordinates of the object in the orbital plane:
\\[P = a(\cos(E) - e)\\]
\\[Q = b\sin(E)\\]

Where a is the semi-major axis and b is the semi-minor axis.

In python:
```python
# Given E, find coordinates in orbital plane (P, Q)
P = semi_major_meters * (math.cos(E) - eccentricity)
Q = semi_minor_meters * math.sin(E)
```

Now, we can convert these into earth-centered cartesian coordinates, with the positive x-axis pointing at the celestial origin. To do this, we need to do three rotations. First we need to rotate around the z-axis by the RAAN, so the ascending node is on the x-axis. Then we need to rotate around the x-axis by the inclination, so the orbit is in the correct plane relative to the equator. Finally, we need to rotate around the z-axis again by the Argument of perigee, so the perigee is in the right position. As an equation, that looks like this:
\\[\vec{\mathbf{r}} = R_z{(-\Omega)}R_x{(-I)}R_z{(-\omega)}\vec{\mathbf{r'}}\\]

\\(R_z\\) and \\(R_x\\) both represent rotation matrices, so written out fully, we get:
\\[\begin{bmatrix} \cos{\Omega} & -\sin{\Omega} & 0 \\\\  \sin{\Omega} & \cos{\Omega} & 0 \\\\ 0 & 0 & 1 \end{bmatrix}
\begin{bmatrix} 1 & 0 & 0 \\\\  0 & \cos{I} & -sin{I} \\\\ 0 & \sin{I} & \cos{I} \end{bmatrix}
\begin{bmatrix} \cos{\omega} & -\sin{\omega} & 0 \\\\  \sin{\omega} & \cos{\omega} & 0 \\\\ 0 & 0 & 1 \end{bmatrix}
\\begin{bmatrix} P \\\\ Q \\\\ 0 \end{bmatrix}\\]

If we multiply all these matrices, we get:
\\[
\begin{bmatrix}
\cos{\omega}\cos{\Omega}-\cos{I}\sin{\omega}\sin{\Omega} & -\cos{\Omega}\sin{\omega}-\cos{I}\cos{\omega}\sin{\Omega} & \sin{I}\sin{\Omega} \\\\\\\\
\cos{I}\cos{\Omega}\sin{\omega} + \cos{\omega}\sin{\Omega} & \cos{I}\cos{\omega}\cos{\Omega}-\sin{\omega}\sin{\Omega} & -\cos{\Omega}\sin{I} \\\\\\\\
\sin{I}\sin{\omega} & \cos{\omega}\sin{I} & \cos{I}
\end{bmatrix}
\begin{bmatrix} P \\\\\\\\ Q \\\\\\\\ 0 \end{bmatrix}
\\]

In python, this looks like the following:
```python
## ROTATION into siderial frame
x = (math.cos(arg_of_perigee_rad) * math.cos(raan_rad) - math.sin(arg_of_perigee_rad) * math.sin(raan_rad) * math.cos(inclination_rad)) * P + \
	(-math.sin(arg_of_perigee_rad) * math.cos(raan_rad) - math.cos(arg_of_perigee_rad) * math.sin(raan_rad) * math.cos(inclination_rad)) * Q

y = (math.cos(arg_of_perigee_rad) * math.sin(raan_rad) + math.sin(arg_of_perigee_rad) * math.cos(raan_rad) * math.cos(inclination_rad)) * P + \
	(-math.sin(arg_of_perigee_rad) * math.sin(raan_rad) + math.cos(arg_of_perigee_rad) * math.cos(raan_rad) * math.cos(inclination_rad)) * Q

z = (math.sin(arg_of_perigee_rad) * math.sin(inclination_rad)) * P + (math.cos(arg_of_perigee_rad) * math.sin(inclination_rad)) * Q
```

## Spherical Coordinates
{% include image.html url="https://upload.wikimedia.org/wikipedia/commons/thumb/4/4f/3D_Spherical.svg/649px-3D_Spherical.svg.png" description="Spherical coordinate system" %}

The last step is to convert the cartestian coordinates into spherical coordinates. The formulas for this transformation are:
\\[r = \sqrt{x^2 + y^2 + z^2}\\]
\\[\theta = \arccos{\frac{z}{r}}\\]
\\[\phi = \arctan{\frac{y}{x}}\\]

Which correspond to radius, altitude and azimuth, respectively. In python, we compute these as follows:
```python
r = math.sqrt(x**2 + y**2 + z**2)
ra_rad = math.atan2(y, x)
dec_rad = math.acos(z / r)
```

We use the atan2 function so we don't have to worry about setting the sign correctly based on the quadrant.

Now we determine the siderial time at Greenwich, which is at zero degrees longitude, using the helper function we wrote before:
```python
gmst_rad = utils.greenwich_siderial_time(time_to_predict) * (2 * math.pi / 24)
```

Siderial time is usually given in hours, so we convert from hours to radians with a simple conversion factor.

To get real longitude, we subtract the local siderial time from the right ascension. You'll also notice in the diagram above that the altitude is defined as the angle between the vector and the north pole. However, latitude is centered at the equator, and ranges from +90 to -90 degrees. Performing both of these conversions is simple, as is converting back to degrees:

```python
lng_rad = -gmst_rad + ra_rad
lat_rad = (math.pi / 2) - dec_rad
if lng_rad > math.pi:
	lng_rad = lng_rad - 2 * math.pi
elif lng_rad < -math.pi:
	lng_rad = lng_rad + 2 * math.pi

lng = math.degrees(lng_rad)
lat = math.degrees(lat_rad)

print("Latitude: %f degrees" % lat)
print("Longitude: %f degrees" % lng)
```

And that's it! We have the latitude of the ISS as a function of time. We can use these numbers to plot ground tracks, or more interestingly, to determine when the ISS is visible to the naked eye, which we'll do in the next post. Stay tuned!
