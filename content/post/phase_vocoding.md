---
date: "2024-01-27"
tags: ["dsp", "mir", "math"]
title: "Phase vocoding"
---

Phase vocoding changes an audio signal's length without influencing its pitch. It also pitch shifts without time stretching audio. The visual counterpart to time stretching is image rescaling. The dimensions of the image would relate to the audio's length, and how the image looks would relate to the audio's pitch. When an image is stretched, its number of pixels would increase. The extra pixels need to be interpolated from its neighbors for the new image to look like the original. We have not achieved perfect interpolations yet, where the audio or image is preserved without side effects. This is noticeable in extreme time stretching, pitch shifting, or image rescaling, and not so much in editing software or media players, since they cap the limits. I'd like to learn more about time stretching as a subproblem to polyphonic melody extraction. That is another unsolved problem where we want to compute the melody when there are concurrent pitches. Extreme time stretching would make studying polyphonic melodies easier and add more tests.

Imagine you sampled a continuous-time signal. Then, you threw every other sample out to halve its length. This would also make its pitch higher unfortunately because the rate at which the signal repeats also doubled. This example is not to be conflated with solely resampling audio. Resampling does not change the length or pitch of audio, but rather makes it more or less clearer. Imagine you sampled a sine wave wherever it repeats, then removed the sine wave but kept the samples. You couldn't tell if the original signal was a straight line or a wave. Therefore, there is a minimum sampling rate to not lose information about the signal, which is twice the highest frequency.

There exist OLA (Overlap Add) methods to time stretch audio in the time domain. Take an audio signal, divide it into overlapping chunks of equal length, and add them together to make the longer audio. However, resulting boundaries between chunks may be discontinuous. To correct for that, you could shift chunks by finding the best offset of the signal. You can do so by using an autocorrelation function, which adds a signal with itself shifted by an offset. The offset in the function resulting closest to the original signal would be the best. However, there may not exist a good offset.

The phase vocoder time stretches audio in the frequency domain. To convert audio to arrays of arrays of frequencies, use an STFT (Short Time Fourier Transform). Kind of like OLA, STFT uses overlapping chunks of equal length, but it also applies an FT to each. To avoid discontinuity, the chunks each use a windowing function, which smooths the beginning and ends of the chunks. With the right windowing function, the inverse fourier transform given the chunks can recreate the original signal. Chunks of waves are represented by arrays of complex numbers, each having magnitudes, which represents energies, and phases, which represents offsets. The index of the arrays would represent the frequencies. The spectrogram is a visual representation of an STFT with color representing the magnitudes, y-axis representing frequencies, and the x-axis representing the nth discrete chunks. If different frequencies have the same phases, they theoretically belong to the same audio source.

To time stretch, you'd interpolate the spectrogram, and to do that, you'd need phase differences. Imagine dividing an audio signal into overlapping chunks. If each chunk contained the same signal which implies each chunk had the same phase, the chunk size would determine the frequency. This works great if the signal repeats itself and isn't chaotic. Therefore, preserving frequencies while interpolating spectrograms would require phase differences or derivatives. Similarly, you could also use multi-band processing. Multi-band processing divides a signal into those with same frequencies before putting them individually in STFTs. Usually, it requires tight divisions for precision. However, transients, which can be sudden percussive sounds or sharp increases in magnitudes, may be lost in phase vocoding, as phase differences do not prefer chaos. If you detect a transient, you can reset the phase difference to the phase of the current chunk.

A Fourier Transform is represented by

\\[ \int_{-\infty}^{\infty} g(t)e^{-2\pi ift} dt \\]

\\( g(t) \\) is the audio signal, represented by sound or air pressure, measured in intensity or voltages over time. If you imagine the repetitive audio signal, centered at the origin and travelling in the path of the unit circle, the rate at which the circle returns to its starting point is the frequency \\( f \\). Euler found that \\( e^{-ti} \\) is the x-coordinate of the point after travelling the distance along the circumference of the unit circle \\( t \\) units starting at \\( (1, 0) \\) counter-clockwise. The frequency \\( f \\) in \\( e^{-2\pi if} \\) slows the rotation around the circle. The center of mass of the centered circular audio signal has two coordinates which are represented with complex numbers. If the center of mass suddenly changes with \\( f \\), this represents a frequency which the original \\( g(t) \\) oscillates at. Calculating the center of mass involves taking the indefinite complex integral of the centered audio signal, which is a precise way of sampling points along the centered signal and averaging them.

The discrete Short Time Fourier Transform (STFT) is

\\[ c(m, n) = \sum\_{l\in \mathbb{Z}} f(l + na_a)g_a(l)e^{-i2\pi ml/M} \\]

where

- \\( m \\) is the FFT length and \\( M \\) is the greatest frequency
- \\( n \\) is the number of STFT frames, hops, or chunks
- \\( l \\) is the index of audio signal \\( f \\), or the analysis signal. \\( L \\) is the window length and not the length of \\( f \\)
- \\( a_a \\) is the analysis time step or hop size
- \\( g_a \\) is a Windowing function. It does not shift leftwards like \\( f(l + na_a) \\) with \\( n \\), ensuring the same windowing strategy is applied to each STFT frame

Choosing the correct window length is important. A window length that is too short does not contain enough frequencies. A window length that is too long has too many conflicting frequencies. The more the windows overlap, the longer the STFTs take. To trade off, you can make window lengths different for different frequencies whose ranges can also be variable. The implementation of this STFT would require recombining multiple FTs.

In the phase vocoder, the phase of successive hops must be preserved from the analysis signal to the synthesis signal, or the output. This is demonstrated by

\\[ \phi_s(m, n) - \phi_s(m, n - 1) = a_s (\Delta_t \phi_a)(m, n) \\]

where 

- \\( \phi_s \\) is the phase of the synthesis signal
- \\( a_s = \alpha a_a \\) is the synthesis hop size and \\( \alpha \\) is the factor to stretch the synthesis signal
- \\( \phi_a = \arg(c(m, n)) \\) is the phase of the analysis signal and \\( \Delta_t \phi_a \\) is the derivative of \\( \phi_a \\) with respect to time
- \\( (\Delta_t\phi_a)(m, n) \approx (\Delta_t\phi_a(m, n - 1) + \Delta_t\phi_a(m, n))/2 \\)
- \\( (\Delta_f\phi_a)(m, n) \\) which differentiates with respect to frequency can substitute \\( (\Delta_t\phi_a)(m, n) \\)

The synthesis phase is used to construct the synthesis signal by way of substituting \\( s(m, n) e^{-i \phi_s} \\) for \\( f \\) back in the STFT, where \\( s(m, n) = \lvert c(m, n) \rvert \\) is the magnitude.

The partial phase derivative with respect to time is approximated with

\\[ \Delta_t \phi_a \approx 1/a_a \left[ \phi_a(m, n) - \phi_a(m, n - 1) - 2\pi ma_a/M \right]_{2\pi} + 2\pi m/M \\]

where \\( \left[ x \right]_{2\pi} = x - 2\pi \lceil \frac{x}{2\pi} \rfloor \\) (the \\( \lceil \rfloor \\)operation rounds to the nearest number) is the principal argument function. Given \\( f(l + na_a) = s(m, n)e^{-i \phi_a} \\) and viewing this substition in the STFT, we need to subtract natural phase shifts from hopping and re-add back the smallest step size to calculate the derivative properly.

The partial phase derivative with respect to frequency is approximated with

\\[ \Delta_f \phi_a \approx 1/b_a \left[ \phi_a(m, n) - \phi_a(m - 1, n) \right]_{2\pi} \\]

where \\( b_a = L/M \\) is the analysis frequency step.

RTPGHI (Real-time Phase Gradient Heap Integration algorithm) then approximates \\( \phi_s(m, n) \\) given \\( \Delta_t \phi_a(m, n), \Delta_t \phi_a(m, n-1), \Delta_t \phi_f(m, n), s(m, n), s(m, n - 1), \phi_s(m, n - 1) \\) and a tolerance variable which was set to \\( 10^{-6} \\).

1. Find the maximum magnitude of the current and previous frame. Multiply this by the tolerance factor to get the abstol value.
2. Let \\( I \\) be the set of all m whose magnitude in the current frame is greater than abstol.
3. Construct a max heap that stores \\( (m, n) \\) tuples, prioritizing them by their magnitudes. Insert \\( (m, n-1) \\) for all \\(m \in I \\).
4. While \\( I \\) is not empty, remove \\( (m_h, n_h) \\) from the top of the heap. 
5. If \\( n_h = n - 1 \\) and \\( (m_h, n_h + 1)\in I \\), calculate \\( \phi_s(m_h, n_h + 1) \\) with the partial time derivative, remove \\( (m_h, n_h + 1) \\) from \\( I \\), and insert it into the heap.
6. If \\( n_h = n \\) and \\( (m_h + 1, n) \in I \\), calculate \\( \phi_s(m_h + 1, n_h) \\) with the partial frequency derivative, remove \\( (m_h + 1, n) \\) from \\( I \\), and insert it into the heap.
7. Do the same thing as 6 but with \\( m_h - 1 \\) instead of \\( m_h + 1 \\).

## Resources

<a id="1">[1]</a>
https://arxiv.org/abs/2202.07382

Original paper

<a id="2">[2]</a>
https://www.youtube.com/watch?v=PjKlMXhxtTM

Phase vocoder background

<a id="3">[3]</a>
https://www.youtube.com/watch?v=n2eW0mugBEs

Background to phase vocoder background

<a id="4">[4]</a>
https://www.youtube.com/watch?v=fJUmmcGKZMI

Phase vocoder background again

<a id="5">[5]</a>
https://www.reddit.com/r/audioengineering/comments/39o46i/why_does_sample_rate_changes_affect_pitch/

Why simplest approach to time stretching doesn't work

<a id="6">[6]</a>
https://www.youtube.com/watch?v=spUNpyF58BY

Fourier transform intuition

<a id="7">[7]</a>
https://www.youtube.com/watch?v=T9x2rvdhaIE

STFT background
