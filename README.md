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
My initial idea was to find a WWVB module that I could simply swap out for a GPS module on an existing Raspberry Pi based NTP server. Universal Solder in Canada sells a WWVB module that seemed to fit the bill, so I ordered one. 	[EverSet ES100 WWVB BPSK Atomic Clock Starter Kit](https://www.universal-solder.ca/product/everset-es100-wwvb-bpsk-atomic-clock-starter-kit/)

Physically connecting it to the Pi was pretty straightforward. It uses I2C to communicate and a couple other GPIO pins for IRQ and Enable. I used the code from [mahtin/es100-wwvb](https://github.com/mahtin/es100-wwvb) to get started. Everything worked, sort of. I was able to get successful time readings from the ES100, and mahtin's code even wrote to SHM so interfacing with NTP was dead simple. However, I quickly understood the limitations of this solution. 

 - I had hoped to get a new reading every minute from the ES100. However, there is a little processing time required, and getting a reading takes about a minute and 20 seconds. So, you're going to get time about every two minutes at best, as far as I can tell.
 - The ES100 has a "tracking" mode that completes a decode in around 30 seconds, so a once-per-minute update seemed feasible. I hacked on the python code for a while and was able to get time updates once a minute, except for the six-minute windows that contain the BPSK extended frames. But, the offset from GPS for those readings was different from the offset I got when decoding a full 60 second frame, and varied from one reading to the next by up to several hundred milliseconds.
 - The ES100 does not decode the 6-minute BPSK extended frames. So, you definitely won't get any readings for 12 minutes out of every hour.
 - I had also hoped to sync to the one second encoded bitstream to update NTP more frequently than once per minute. But, the ES100 does not expose the PWM or BPSK encoded bitstreams at all. You set the enable pin on your GPIO and wait a minute and a half, and maybe you will get the time. Maybe not.

I consider this first attempt to be a partial success, but the results were a bit underwhelming. I was getting time, but it varied by tens or hundreds of milliseconds from one reading to the next. It's certainly good enough for building a digital clock display that sets the time automatically. But, it is definitely not a time source you want to use for an NTP reference clock. For my purpose, I need direct access to the WWVB signal itself.

# Second Attempt
After experimenting with the ES100 and understanding its limitations, I set out to find a receiver that would give me access to the raw WWVB signal. Finding nothing ready-made, I realized I would have to build an RF front end and use an analog-to-digital-converter (ADC) to get direct access to the data stream. I found this fantastuic [blog post](https://www.burningimage.net/clock/sensitive-60khz-receiver/) with a diagram for a suitable RF front end, tuned specifically to receive a 60KHz WWVB signal. Designign and building electronic circuit from components is way outside my realm of expertise, but I buoght a breadboard and some assorted electronics parts and set out to build a receiver. If you already understanf voltage dividers and op-amps and low-pass filters you will be light years ahead of me. But I managed to get the circuit laid out on a breadboard and start testing.


