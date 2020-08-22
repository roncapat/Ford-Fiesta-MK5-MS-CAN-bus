# Reverse engineering CANbus messages of Fiesta MK5 dashboard/radio subsystem
This documentation applies to Ford Fiesta MK5 >2006 (MK5 restyling), with 2-DIN, 6000CD radio.

The radio sends messages to the LCD screen in the dashboard, like FM station names, CD track IDs, volume changes.

The idea was to tap into the text-to-LCD mechanism, but I ended up reversing some other interesting CANbus messages.

Tapping in the infotaiment CANbus is quite easy: just connect a CANbus interface over CANH and CANL lines that come into the QuadLock conector of the radio, no 120 Ohm terminal block resistor needed.

![QuadLock connector and CANbus pins](quadlock.png)

Personally, I used a [SeedStudio CANbus Shield](https://wiki.seeedstudio.com/CAN-BUS_Shield_V2.0/) stacked onto an Arduino Uno R3. 

## Results
Bus speed is 125 KBPS.

### Text to LCD screen
The radio sends text to the dashboard LCD using either a short message format or an extended message format. 

The short format can only be seen whenever the radio switches to AUX mode.

<table class="tg">
<thead>
  <tr>
    <th class="tg-c3ow">ID</th>
    <th class="tg-c3ow">1</th>
    <th class="tg-c3ow">2</th>
    <th class="tg-c3ow">3</th>
    <th class="tg-c3ow">4</th>
    <th class="tg-c3ow">5</th>
    <th class="tg-c3ow">6</th>
    <th class="tg-c3ow">7</th>
    <th class="tg-c3ow">8</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-c3ow">0x4C0</td>
    <td class="tg-c3ow">0x7</td>
    <td class="tg-c3ow">0x34</td>
    <td class="tg-c3ow" colspan="6">ASCII text</td>
  </tr>
</tbody>
</table>

The extended format is composed by one header message containing the length of the text, and subsequent messages that provide additional text portions.

<table class="tg">
<thead>
  <tr>
    <th class="tg-c3ow">ID</th>
    <th class="tg-c3ow">1</th>
    <th class="tg-c3ow">2</th>
    <th class="tg-c3ow">3</th>
    <th class="tg-c3ow">4</th>
    <th class="tg-c3ow">5</th>
    <th class="tg-c3ow">6</th>
    <th class="tg-c3ow">7</th>
    <th class="tg-c3ow">8</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-c3ow">0x4C0</td>
    <td class="tg-c3ow">0x10</td>
    <td class="tg-c3ow">Text length + 2</td>
    <td class="tg-c3ow">0x34</td>
    <td class="tg-c3ow" colspan="5">ASCII text</td>
  </tr>
  <tr>
    <td class="tg-0pky">0x4C0</td>
    <td class="tg-0pky">0x21</td>
    <td class="tg-0pky" colspan="7">ASCII text</td>
  </tr>
</tbody>
</table>

### Radio OFF
When the radio is OFF this message is broadcasted.
<table class="tg">
<thead>
  <tr>
    <th class="tg-c3ow">ID</th>
    <th class="tg-c3ow">1</th>
    <th class="tg-c3ow">2</th>
    <th class="tg-c3ow">3</th>
    <th class="tg-c3ow">4</th>
    <th class="tg-c3ow">5</th>
    <th class="tg-c3ow">6</th>
    <th class="tg-c3ow">7</th>
    <th class="tg-c3ow">8</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-c3ow">0x4C0</td>
    <td class="tg-c3ow">0x2</td>
    <td class="tg-c3ow">0x34</td>
    <td class="tg-c3ow" colspan="6">0</td>
  </tr>
</tbody>
</table>

### Date & Time
The car has an onboard RTC that works independently from the radio clock (it works even if the radio is disconnected at all). The date and time are broadcasted with 1 second resolution.
<table class="tg">
<thead>
  <tr>
    <th class="tg-c3ow">ID</th>
    <th class="tg-c3ow">1</th>
    <th class="tg-c3ow">2</th>
    <th class="tg-c3ow">3</th>
    <th class="tg-c3ow">4</th>
    <th class="tg-c3ow">5</th>
    <th class="tg-c3ow">6</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-c3ow">0x80</td>
    <td class="tg-c3ow">year (two digits)</td>
    <td class="tg-c3ow">month</td>
    <td class="tg-c3ow">day</td>
    <td class="tg-c3ow">hours</td>
    <td class="tg-c3ow">minutes</td>
    <td class="tg-c3ow">seconds</td>
  </tr>
</tbody>
</table>

### AirBag status 

<img src="K01_0.png" data-canonical-src="K01_0.png" width="50"/>

The AirBag dashboard light is commanded either ON or OFF depending with the 5th byte of this message. 

<table class="tg">
<thead>
  <tr>
    <th class="tg-c3ow">ID</th>
    <th class="tg-c3ow">1</th>
    <th class="tg-c3ow">2</th>
    <th class="tg-c3ow">3</th>
    <th class="tg-c3ow">4</th>
    <th class="tg-0lax">5</th>
    <th class="tg-0lax">6</th>
    <th class="tg-c3ow">7</th>
    <th class="tg-c3ow">8</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-c3ow">0x460</td>
    <td class="tg-c3ow">0x0</td>
    <td class="tg-c3ow">0x0</td>
    <td class="tg-c3ow">0x0</td>
    <td class="tg-c3ow">0x0</td>
    <td class="tg-0lax">&lt;status&gt;</td>
    <td class="tg-0lax">0x0</td>
    <td class="tg-c3ow">0x0</td>
    <td class="tg-c3ow">0x0</td>
  </tr>
</tbody>
</table>

<table class="tg">
<thead>
  <tr>
    <th class="tg-c3ow">Status</th>
    <th class="tg-c3ow">Code</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-c3ow">ON</td>
    <td class="tg-c3ow">0xC0</td>
  </tr>
  <tr>
    <td class="tg-c3ow">OFF</td>
    <td class="tg-c3ow">0x0</td>
  </tr>

</tbody>
</table>

### Doors status
Doors status is signalled setting or clearing bits in the 1st byte of this message.

<table>
<thead>
  <tr>
    <th>ID</th>
    <th>1</th>
    <th>2</th>
    <th>3</th>
    <th>4</th>
    <th>5</th>
    <th>6</th>
    <th>7</th>
    <th>8</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>0x433</td>
    <td>&lt;status&gt;</td>
    <td colspan="7">(unspecified)</td>
  </tr>
</tbody>
</table>

Status is a bitmask: the 1st byte is the algebraic sum of these condition codes, whenever applicable.

<table>
<thead>
  <tr>
    <th>Status</th>
    <th>Code</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Front left open</td>
    <td>0x80</td>
  </tr>
  <tr>
    <td>Front right open</td>
    <td>0x40</td>
  </tr>
  <tr>
    <td>Trunk open</td>
    <td>0x08</td>
  </tr>
</tbody>
</table>


### Arrows status

<img src="H34.png" data-canonical-src="H34.png" width="50"/>

Arrows status is signalled setting or clearing bits in the 1st byte of this message.

<table>
<thead>
  <tr>
    <th>ID</th>
    <th>1</th>
    <th>2</th>
    <th>3</th>
    <th>4</th>
    <th>5</th>
    <th>6</th>
    <th>7</th>
    <th>8</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>0x265</td>
    <td>&lt;status&gt;</td>
    <td colspan="7">(unspecified)</td>
  </tr>
</tbody>
</table>

Status is a bitmask: the 1st byte is the algebraic sum of these condition codes, whenever applicable.

<table>
<thead>
  <tr>
    <th>Status</th>
    <th>Code</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Left ON</td>
    <td>0x20</td>
  </tr>
  <tr>
    <td>Right ON</td>
    <td>0x40</td>
  </tr>
</tbody>
</table>


### Vehicle ID
Periodically, the veihcle ID (lower part of VIN, so the serial number plus other codes signalling production date) is broadcasted.

<table>
<thead>
  <tr>
    <th>ID</th>
    <th>1</th>
    <th>2</th>
    <th>3</th>
    <th>4</th>
    <th>5</th>
    <th>6</th>
    <th>7</th>
    <th>8</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>0x4F3</td>
    <td colspan="8">&lt;vehicle ID&gt;</td>
  </tr>
</tbody>
</table>

### Front beams status

<img src="H35.png" data-canonical-src="H35.png" width="50"/> <img src="H36.png" data-canonical-src="H36.png" width="50"/>

Front beams status is signalled setting or clearing bits in the 1st byte of this message. Notice that, since this message is just for dashboard LEDs, parking lights cannot be distinguished from low beams, as they share the same LED indicator.

<table>
<thead>
  <tr>
    <th>ID</th>
    <th>1</th>
    <th>2</th>
    <th>3</th>
    <th>4</th>
    <th>5</th>
    <th>6</th>
    <th>7</th>
    <th>8</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>0x286</td>
    <td>&lt;0x10+status&gt;</td>
    <td colspan="7">(unspecified)</td>
  </tr>
</tbody>
</table>

Status is a bitmask: the 1st byte is the algebraic sum of 0x10 and these condition codes, whenever applicable.
<table>
<thead>
  <tr>
    <th>Status</th>
    <th>Code</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Parking lights / low beams</td>
    <td>0x80</td>
  </tr>
  <tr>
    <td>High beams</td>
    <td>0x40</td>
  </tr>
</tbody>
</table>
