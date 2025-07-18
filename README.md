# WWVB based NTP server
GPS is the de-facto standard for NTP reference clocks. Inexpensive GPS modules can easily be added to a Raspberry Pi in order to build an accurate reference clock for NTP. I have been wanting to do something a bit different and utilize an older technology to obtain accurate timing for an NTP server. I am going to use WWVB as a time source for an NTP reference clock. No, it's not as accurate as GPS. No, it's nto practical. It's just for fun. And it turns out not to be as straighforward as I first imagined.

# WWVB
If you are not familiar with WWVB, it is an AM radio signal broadcast out of Fort Collins, Colorado, and is operated by the National Institute of Standards and Technology (NIST). See: https://www.nist.gov/pml/time-and-frequency-division/time-distribution/radio-station-wwvb

WWVB is commonly used in inexpensive radio-controlled clocks in the United States. The clock includes a small ferrite rod antenna and a chip to demodulate the AM signal and automatically set the time. I have owned a few of these over the years, and they work pretty well if you place them in a location that gets a good signal from the WWVB broadcast. In my experience, proper placement and orientation can be a fiddly process. These clocks typically attempt readio reception at night when the amount of RF interference is lowest and update the local timing circuit. They are not typically designed for continuous reception like I want in a time server.

# WWVB Modulation
## Amplitude modulation
WWVB uses a pulse width modulation scheme on the AM transmission to encode time data. Every minute, sixty one-second bits of data are encoded by reducing the amplituded of the carrier signal for a specific interval. 0.2s for a 0 bit, 0.5s for a 1 bit, and 0.8s for a framing marker. These sixty bits are used to encode the time and date into a one minute data frame that can be decoded by a simple receiver.

## Phase Modulation
Starting in 2012, WWVB added phase modulation to the signal in order to encode additional data on the 60KHz carrier. Binary phase shift keying (BPSK) is used to encode the time and date alongside the PWM data. Twice per hour, this BPSK modulation is used to send an extended six-minute data frame which can be used to provide a more robust and reliable method of detecting the potentially weak WWVB signal for any receiver circuits that are able to decode it.

# First attempt
My initial idea was to find a WWVB module that I could simply swap out for a GPS module on an existing Raspberry Pi based NTP server. Universal Solder in Canada makes a WWVB module that seemed to fit the bill, so I ordered one. 	[EverSet ES100 WWVB BPSK Atomic Clock Starter Kit](https://www.universal-solder.ca/product/everset-es100-wwvb-bpsk-atomic-clock-starter-kit/)
