## Simulator Developed for Lab 4
When we first finished our `CONTROL_UNIT` and connected it to the camera, we got gibberish images like this:

![img_4412](https://user-images.githubusercontent.com/42748229/48320204-c417d300-e5e4-11e8-823f-b81499c75236.jpg)

Since we have an integrated system of different components, we had a hard time isolating and pinpointing the reason why this is happening. Therefore, we thought that we could create a module that simulates camera outpus to test our `CONTROL_UNIT`. This is the definition of the module:

```verilog
`define SCREEN_WIDTH 176
`define SCREEN_HEIGHT 144
`define STRIDE 180
`define FRAME_LENGTH 800000 //0 VSYNC low, 1-2 VSYNC high, 3 VSYNC low, then start at 4, each row with 176*2 valid data bytes while HREF high, then 8 cycles while HREF low

module SIMULATOR(
  CLK,
  VSYNC,
  HREF,
  DATA
);
```

Note that the camera module is not outputting a clock, as this is the ideal simulation when `CONTROL_UNIT` and camera are using the same clock. When actually using a camera, we had to make sure that we connected the correct clock signal. The source code of the veilog file can be found [here](https://github.com/ECE3400-Team14/Lab4_FPGA/blob/master/Lab4_FPGA_Template/SIMULATOR.v).

It is pretty straightforward. There are two always blocks. One controls `VSYNC`, `HREF`, and which byte of the pixel that the module is suppoed to be outputting. The other controls the color of a pixel determined by the X and Y addresses.

Using this simulator, we managed to use our `CONTROL_UNIT` to get real images. However, we encountered problems regarding colors that we did not see while using our simulator. Therefore, keep in mind that this is merely an ideal simulation.
