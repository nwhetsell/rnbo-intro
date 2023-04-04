# RNBO

[RNBO](https://rnbo.cycling74.com) is a technology for creating audio apps (like synths, sample-based instruments, and audio effects) in a [Max](https://cycling74.com/products/max)-like [Patcher window](https://docs.cycling74.com/latest/vignettes/patcher_window_navigation) and exporting them to a variety of formats (like [VSTs](https://en.wikipedia.org/wiki/Virtual_Studio_Technology), [Audio Units](https://en.wikipedia.org/wiki/Audio_Units), [WebAssembly](https://en.wikipedia.org/wiki/WebAssembly), and C++ code).

RNBO is pronounced the same as the word _rainbow_. [Cycling&nbsp;’74](https://cycling74.com) (the developer of Max and RNBO) has not disclosed what, if anything, the acronym _RNBO_ stands for.

It’s probably best to think of RNBO as a patching environment that looks like Max, but is separate from Max. This is analogous to how [Gen](https://docs.cycling74.com/latest/vignettes/gen_topic) works. When you create a [gen~](https://docs.cycling74.com/latest/refpages/gen~) object, hold the pointer over the left side of the object and click the [object action button](https://docs.cycling74.com/latest/vignettes/inspector) (looks like ► enclosed in a circle), and then choose Edit from the menu that appears, you open a [Gen patcher window](https://docs.cycling74.com/latest/vignettes/gen_overview#The_Gen_Patcher_Window). A Gen patcher window is similar to a Max patcher window, but there are [important differences](https://docs.cycling74.com/latest/vignettes/gen_overview#Patching_in_Gen). For example, there are no integers in gen~; [there are only floating-point numbers](https://docs.cycling74.com/latest/vignettes/gen_overview#Technical_notes).

As with Gen, you use RNBO by creating a [rnbo~](https://docs.cycling74.com/latest/refpages/rnbo~) object in a Max patcher window. However, RNBO is licensed separately from Max; you must [purchase a RNBO license](https://cycling74.com/shop/rnbo) before you can save RNBO patches.

## Getting Started with RNBO

Most people will probably get started with RNBO by opening Cycling&nbsp;’74’s RNBO intro patch, either from the [RNBO Quickstart](https://rnbo.cycling74.com/learn/quickstart) or by choosing Extras&nbsp;> RNBO Intro in Max. This patch may seem overwhelming, but breaking it down will help us understand what RNBO is and how to use it.

### start automation ①, adjust gain ②

When we enable the toggle labeled “start automation”, two things happen:

1. Four groups of four notes begin playing repeatedly
2. Three attributes—cutoff, harmonics, and overblow—begin varying

These things are controlled by the Automation subpatcher. In this subpatcher, notes are produced by a [metro](https://docs.cycling74.com/latest/refpages/metro) that bangs once every 1500&nbsp;ms (40&nbsp;bpm). On each bang, a [counter](https://docs.cycling74.com/latest/refpages/counter) outputs the integers 0–3 in succession. (The [trigger](https://docs.cycling74.com/latest/refpages/trigger) that follows appears to be unnecessary; only one of its outputs is used.) The output of the counter is used to select one of four messages that each contains a list of four numbers; these are the MIDI note numbers of the notes that are playing:

| MIDI Note Numbers | Pitches     |
| ----------------- | ----------- |
| 42 57 62 64       | F♯ A D E    |
| 49 50 64 69       | C♯ D E A    |
| 40 50 55 62       | E D G D     |
| 38 52 55 67       | D E G G     |

These 4-number lists are sent to a collection of objects that delays each number by integer multiples of 200&nbsp;ms. The [listfunnel](https://docs.cycling74.com/latest/refpages/listfunnel) object is similar to [iter](https://docs.cycling74.com/latest/refpages/iter), but listfunnel prepends each list item’s index to its output (listfunnel sends a succession of two-item lists). The order of listfunnel’s output is then reversed using [`$` notation](https://docs.cycling74.com/latest/vignettes/dollar_sign_and_pound_sign) in a message box, [unpack](https://docs.cycling74.com/latest/refpages/unpack)’d, and finally sent to [pipe](https://docs.cycling74.com/latest/refpages/pipe) after multilpying each index by 200. The result is that MIDI note numbers are sent after delays of 0&nbsp;ms for the first one (it’s sent immediately), then 200&nbsp;ms, then 400&nbsp;ms, and finally 600&nbsp;ms for the last one. This approximates a succession of four 32nd notes (multiplying indexes by 1500&nbsp;ms ÷ 8 = 187.5&nbsp;ms would’ve achieved 32nd notes exactly). Finally, note numbers are sent to [makenote](https://docs.cycling74.com/latest/refpages/makenote) with a [random](https://docs.cycling74.com/latest/refpages/random) velocity between 90 and 101.

Automation of cutoff, harmonics, and overblow is much more straightforward. This is controlled by sine waves ([cycle~](https://docs.cycling74.com/latest/refpages/cycle~), for cutoff and overblow) or a saw tooth wave ([phasor~](https://docs.cycling74.com/latest/refpages/phasor~), for harmonics) that are sampled every 10&nbsp;ms and then scaled to an appropriate range. (Note the use of integers in the scaling of harmonics; using integers ensures that the value of harmonics can be 1 or 2, for example, but not 1.5.) The remainder of the Automation subpatcher ensures that the onscreen keyboard ([kslider](https://docs.cycling74.com/latest/refpages/kslider)) in the main patcher can’t be clicked while it’s automated.

### Double click to open ③

In the main patcher, the [rnbo~](https://docs.cycling74.com/latest/refpages/rnbo~) object is instantiated with the attributes “polyphony” and “title”. Both attributes are optional, but this touches on a major difference between RNBO and Max: polyphony in RNBO is [greatly simplified](https://rnbo.cycling74.com/learn/polyphony-and-voice-management-in-rnbo#simple-polyphony), and RNBO has no [poly](https://docs.cycling74.com/latest/refpages/poly) object. To make a RNBO patch with 8-voice polyphony, you simply type “@polyphony 8” (as shown in the intro patch).

Seven [attrui](https://docs.cycling74.com/latest/refpages/attrui) objects are used to set additional attributes of rnbo~, and at first glance these attributes may seem unusual. Not all audio effects will include a filter, yet rnbo~ objects appear to have “cutoff” and “Q“ attributes. When we double-click on the rnbo~ object to open its RNBO patch window, we can begin to see what’s going on. The RNBO patch window contains four [param](https://rnbo.cycling74.com/objects/ref/param) objects, like this:

1. param overblow @value 1.5 @min 0.1 @max 5
2. param harmonics @value 3 @min 0.1 @max 10
3. param cutoff @value 880 @min 100 @max 8000 @exponent 2
4. param Q @value 3 @min 0.01 @max 10

These correspond to four of the attrui objects in the main patch. Among other things, param objects can set parameter names, default values (@value), minimums and maximums (@min and @max), and (in the case of cutoff) an exponential mapping (@exponent). This is one of the major design patterns of RNBO: you declare parameters in a RNBO patch, and these parameters are exposed to Max as attributes. In the intro patch, RNBO parameters are sent to [slide~](https://rnbo.cycling74.com/objects/ref/slide~) objects, which output numbers at audio rate after logarithmically smoothing them.

There is a “gotcha” in this RNBO patch window. In the main patcher, the rnbo~ object’s rightmost inlet receives MIDI notes. The usual way of creating inlets in subpatchers is by using [inlet](https://docs.cycling74.com/latest/refpages/inlet) objects, but no inlet objects appear in the RNBO patcher. This is because RNBO has no inlet objects. Instead, when you add [certain MIDI input objects](https://rnbo.cycling74.com/learn/midi-in-rnbo#adding-midi-input) (like [notein](https://rnbo.cycling74.com/objects/ref/notein)) to a RNBO patch, RNBO automatically creates a MIDI input as the rightmost inlet. (You can add additional inlets to a rnbo~ object by using [in](https://rnbo.cycling74.com/objects/ref/in) objects.)

The MIDI note’s frequency, overblow parameter, and harmonics parameter are each input to a [gen~](https://rnbo.cycling74.com/objects/ref/gen~) object. This illustrates another point: [Gen is available in RNBO](https://rnbo.cycling74.com/learn/gen-rnbo). This particular gen~ object is probably the most important one in the intro patch, so we should take a look at it.

### Gen in RNBO

The Gen patch may look inscrutable at first, so let’s take it piece by piece. As an initial matter, it’s important to note that [Gen patches operate on individual samples](https://cycling74.com/tutorials/gen~-for-beginners-part-1-a-place-to-start). (This is why there are no tildes (~) after Gen object names; everything is audio-rate, so tildes would be redundant.) For now, pretend that the [accum](https://docs.cycling74.com/latest/refpages/gen_dsp_accum) object outputs directly to [sin](https://docs.cycling74.com/latest/refpages/gen_common_sin), which outputs directly to [out](https://docs.cycling74.com/latest/refpages/gen_common_out)&nbsp;1. In this scenario, the Gen patch would output a sine wave. To see this, note that initially the accum object stores 0. Remembering that MIDI note frequency (let’s call this $f$) is input to [in](https://docs.cycling74.com/latest/refpages/gen_common_in)&nbsp;1, after one sample, accum will store

$$
\pi \times f / (F_s / 2) = 2 \pi f \times 1/F_s
$$

where $F_s$ is the sampling rate (so $1/F_s$ is the sampling period). This number is the _instantaneous phase_—often shortened to just _phase_—of a sine wave. Taking the sine of this phase gives the value of a sine wave at $t = 1/F_s$, that is, the value of a sine wave after 1 sampling period. After two samples, another $2 \pi f \times 1/F_s$ is added to the accum object, so it will store a phase of $2 \pi f \times 2/F_s$; taking the sine of that phase gives the value of a sine wave after 2 sampling periods. After three samples, the accum object will store a phase of $2 \pi f \times 3/F_s$; taking the sine of that phase gives the value of a sine wave after 3 sampling periods; and so on. After many sampling periods, we’ll hear a sine wave.

Now let’s try to make sense of the [+](https://docs.cycling74.com/latest/refpages/gen_common_add) object that precedes the sin object. The output of the sin object is sent to a [history](https://docs.cycling74.com/latest/refpages/gen_dsp_history) object; this is a 1-sample delay that Gen uses to support feedback loops. There is some additional plumbing—two gen objects titled “lowpass” that we’ll come back to in a moment, followed by a [*](https://docs.cycling74.com/latest/refpages/gen_common_mul) object—but the output of the sin object is being added to—_modulating_—the sin object’s own phase. This is _feedback frequency modulation_, and indeed the title of the gen~ object in the RNBO patcher window is “feedback-fm”. (Technically, we’re modulating phase and not frequency, but phase modulation and frequency modulation [are mathematically equivalent](https://ccrma.stanford.edu/software/snd/snd/fm.html).)

Now let’s look at one of the two gen objects titled “lowpass” (the internals of these objects are identical, so it doesn’t matter which we choose). Here, we see a [codebox](https://docs.cycling74.com/latest/refpages/gen_common_codebox) object (which is confusingly styled as `CodeBox` in the object itself) containing [GenExpr](https://docs.cycling74.com/latest/vignettes/gen_genexpr) code. The codebox has two inputs (for cutoff frequency and Q) and five outputs. Practically all the code in the codebox is for performing calculations, which may be easier to see if we simplify the code and use more descriptive variable names:

```javascript
omega = cutoff_frequency * twopi / samplerate;
cos_omega = cos(omega);
alpha = sin(omega) * 0.5 / Q;

b0 = 1 / (1 + alpha);
a2 = (1 - cos_omega) * 0.5 * b0;
a0 = a2;
a1 = (1 - cos_omega) * b0;
b1 = -2 * cos_omega * b0;
b2 = (1 - alpha) * b0;

out1 = a0;
out2 = a1;
out3 = a2;
out4 = b1;
out5 = b2;
```

This may still look like rocket science, but it turns out that this is just calculating some numbers (_biquad coefficients_) needed to perform a standard type of lowpass filtering described in the [W3C’s _Audio EQ Cookbook_](https://www.w3.org/TR/audio-eq-cookbook/). Equations of filters are often [written as](https://www.w3.org/TR/audio-eq-cookbook/#MathJax-Element-6-Frame)

$$
\begin{align}
y[n] = \left(\frac{b_0}{a_0}\right)x[n] & + \left(\frac{b_1}{a_0}\right)x[n-1] + \left(\frac{b_2}{a_0}\right)x[n-2] \\
  & - \left(\frac{a_1}{a_0}\right)y[n-1] - \left(\frac{a_2}{a_0}\right)y[n-2]
\end{align}
$$

This means that to compute the output at sample $n$, we take the input at sample $n$ ($x[n]$), add the input from 1 and 2 samples ago, subtract the output from 1 and 2 samples ago, and multiply everything by various coefficients (for example $\frac{b_0}{a_0}$). The task, then, is to determine what these coefficients are. The _Audio EQ Cookbook_ [includes](https://www.w3.org/TR/audio-eq-cookbook/#MathJax-Element-28-Frame) the exact same calculations as the first few lines of the codebox:

$$
\begin{align}
\omega_0 &= f_0 \times 2 \pi / F_s = 2 \pi \frac{f_0}{F_s} \\
\alpha &= \sin \omega_0 \times 0.5 / Q = \frac{\sin \omega_0}{2 Q}
\end{align}
$$

The _Audio EQ Cookbook_ then [gives](https://www.w3.org/TR/audio-eq-cookbook/#LPF-eqn) the numerators and denominators of the coefficients as:

$$
\begin{align}
b_0 &= \frac{1 - \cos \omega_0}{2} \\
b_1 &= 1 - \cos \omega_0 \\
b_2 &= \frac{1 - \cos \omega_0}{2} \\
a_0 &= 1 + \alpha \\
a_1 &= -2 \cos \omega_0 \\
a_2 &= 1 - \alpha
\end{align}
$$

(Note that $b_0$ and $b_2$ are equal.) The outputs of the codebox correspond to these numerators and denominators:

$$
\begin{align}
\texttt{out1} &= \texttt{a0} = \texttt{a2} = \frac{1 - \cos \omega_0}{2} \times \texttt{b0} = \frac{1 - \cos \omega_0}{2} \times \frac{1}{1 + \alpha} = \frac{b_0}{a_0} \\
\texttt{out2} &= \texttt{a1} = (1 - \cos \omega_0) \times \texttt{b0} = (1 - \cos \omega_0) \times \frac{1}{1 + \alpha} = \frac{b_1}{a_0} \\
\texttt{out3} &= \texttt{a2} = \frac{b_0}{a_0} = \frac{b_2}{a_0} \\
\texttt{out4} &= \texttt{b1} = -2 \cos \omega_0 \times \texttt{b0} = -2 \cos \omega_0 \times \frac{1}{1 + \alpha} = \frac{a_1}{a_0} \\
\texttt{out5} &= \texttt{b2} = (1 - \alpha) \times \texttt{b0} = (1 - \alpha) \times \frac{1}{1 + \alpha} = \frac{a_2}{a_0}
\end{align}
$$

(Unfortunately, Cycling&nbsp;’74 chose very confusing variable names. `a0`, `a1`, `a2`, `b0`, `b1`, and `b2` in the codebox are _not_ the same as $a_0$, $a_1$, $a_2$, $b_0$, $b_1$, and $b_2$ in the _Audio EQ Cookbook_.) These outputs are sent to yet another gen object titled “biquad” that performs filtering with the coefficients calculated in the codebox.

Returning to the feedback-fm Gen patcher window, we can now see how the overblow (passed as in&nbsp;2) and harmonics (in&nbsp;3) parameters are used. The “overblow” parameter is really a gain applied to the output of the lowpass filters before it’s used for phase modulation. The “harmonics” parameter is used to calculate the cutoff frequency of the lowpass filters (by simply multiplying by the input frequency). (Recall that the harmonics parameter was restricted to integers in the Automation subpatcher. Since “harmonics” is actually a filter cutoff frequency, this restriction is unnecessary; the value of “harmonics” can smoothly vary without introducing inharmonic frequencies.)

As a final step, the output is multiplied by 10 and sent to a [tanh](https://docs.cycling74.com/latest/refpages/tanh) object. This is a quick-and-dirty overdrive effect; you may want to experiment with the MSP [tanh~](https://docs.cycling74.com/latest/refpages/tanh~) reference patch, which applies the same effect to a sine wave.

### RNBO Subpatchers

The rest of the RNBO patch consists of [subpatchers](https://rnbo.cycling74.com/objects/ref/patcher), which are largely identical to subpatchers in Max. The first subpatcher uses [adsr~](https://rnbo.cycling74.com/objects/ref/adsr~) to apply a standard [ADSR envelope](https://en.wikipedia.org/wiki/Envelope_(music)#ADSR) to notes. Note velocity, which is 0 when a note stops and 1–127 otherwise, is used to trigger the envelope. The second subpatcher uses [filtercoeff~](https://rnbo.cycling74.com/objects/ref/filtercoeff~) and [biquad~](https://rnbo.cycling74.com/objects/ref/biquad~) to apply a lowpass filter (this is very likely doing the same calculations as the Gen subpatch, but with RNBO objects).

The last subpatcher applies a stereo delay and is worth spending a little more time with. This subpatcher contains additional param objects, and these appear in the main patch as attributes with names like “poly/delay/left_delay”. The “left_delay” part of the name is, of course, the name specified in the param object. The “delay/” part of the name is from the patcher object’s scripting name (_not_ the subpatcher name) and is accessible from the [object Inspector](https://docs.cycling74.com/latest/vignettes/inspector). Finally, parameters declared in RNBO subpatchers are prefixed with “poly/”. This seems to be because RNBO affords a lot of flexibility with subpatcher polyphony. A RNBO subpatcher’s polyphony need not be the same as the main RNBO patcher’s, and it’s possible to [design a polyphonic patch with monophonic components](https://rnbo.cycling74.com/learn/polyphony-and-voice-management-in-rnbo).

The actual implementation of the delay may look confusing, but this is mostly due to how the subpatch is laid out. This is really two copies of the reference patch for RNBO’s [feedback~](https://rnbo.cycling74.com/objects/ref/feedback~) object, with a few extra objects for parameter management and applying attenuation.

## Exporting RNBO Patches

Now that we understand the RNBO patch, let’s export it so that we can use it outside of Max.

### Exporting an Audio Unit

First, we’ll export the RNBO patch as an Audio Unit. To do this, click the Export Sidebar button (looks like a document with →) to open the [Export Sidebar](https://rnbo.cycling74.com/learn/rnbo-export-sidebar-overview), and then double-click Audio Plugin Export. Select “AU Mac” from the Format and Platform menu, and choose

```
~/Library/Audio/Plug-Ins/Components
```

for the Output Directory (one way to do this is to press the Choose button, press Command-Shift-G, and paste that directory into the Go to Folder sheet that appears). Note the Plugin Manufacturer Name: “My Company”. Press the Export to Selected Target button at the bottom right of the Export Sidebar; after a few seconds, the export should complete. You should now be able to open a DAW of your choice, find the RNBO plug-in under “My Company”, and use it as an instrument.

In the plug-in’s UI, there’s a Preset menu with items named “smooth”, “bright”, and “nasty”. You can create presets for RNBO patches, but these are stored as [Max snapshots](https://rnbo.cycling74.com/learn/presets-with-snapshots); RNBO doesn’t have [pattr](https://docs.cycling74.com/latest/refpages/pattr) objects like Max.

### Exporting WebAssembly

Finally, let’s export the RNBO patch as WebAssembly that can run in a webpage. To do this, you’ll need to [create a GitHub account](https://github.com/signup), and then create a copy of [Cycling&nbsp;’74’s RNBO website repository](https://github.com/Cycling74/rnbo.example.webpage) by clicking “Use this template” > “Create a new repository”. Download this repository to your computer by using the free [GitHub Desktop](https://desktop.github.com) app, and then open a [Terminal](https://support.apple.com/guide/terminal/open-or-quit-terminal-apd5265185d-f365-44cb-8b09-71a064a42125/mac) (or [PowerShell](https://learn.microsoft.com/en-us/powershell/) on Windows) to the repository folder.

Before continuing, you should install [Node.js](https://nodejs.org) (you only need to do this once). The simplest way to do this on Mac is to install [Homebrew](https://brew.sh) and then run

```sh
brew install node
```

On Windows, you can use [Chocolatey](https://community.chocolatey.org/packages/nodejs).

In RNBO’s Export Sidebar, double-click Web Export. Choose the “export” subfolder in the repository folder as the Output Directory. Note the File Name: feedback-synth.export.json. Press the Export to Selected Target button at the bottom right. In Terminal (or PowerShell), run:

```sh
npx http-server -c-1
```

(The `-c-1` argument disables caching of JavaScript files by the browser.) You may be prompted to install the [http-server Node.js pacakge](https://www.npmjs.com/package/http-server), but eventually you should see output like:

```
Available on:
  http://127.0.0.1:8080
  http://192.168.1.206:8080
```

Paste either URL from your output (your URLs may not be the same as the ones above) into your browser’s address field to open your RNBO app’s website. The first time you do this, you should see an error message. This is because the website is trying to load export/patch.export.json (which doesn’t exist) instead of the correct export/feedback-synth.export.json. To fix this, open [js/app.js](https://github.com/Cycling74/rnbo.example.webpage/blob/main/js/app.js) in a text editor, and change line&nbsp;2 from

```javascript
    const patchExportURL = "export/patch.export.json";
```

to

```javascript
    const patchExportURL = "export/feedback-synth.export.json";
```

In Terminal (or PowerShell), press Control-C to stop the server, rerun `npx http-server -c-1` to restart it, and then reload the RNBO app’s website. The website should now load normally; try clicking the buttons of the “MIDI keyboard” to hear the RNBO synth.

## Further Reading

This is really just scratching the surface of RNBO. Here are two resources where you can learn more:

* https://rnbo.cycling74.com/learn
* https://rnbo.cycling74.com/explore
