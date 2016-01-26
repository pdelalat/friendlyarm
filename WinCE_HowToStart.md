Running Windows CE 6.0 on mini2440 is not very hard. You just have to download the BSP of the board, install software development tools (free evaluation version available on the Microsoft website), generate a Windows CE image and flash it in the board.
When Windows CE runs on the mini2440, it is then possible to develop Windows CE applications using Visual Studio 2005 pro.

The following links will guide you through all these stages, from the generation of the Windows CE image to a first application development:

  * Development tools installation, WinCE image generation ([EnglishBSP](EnglishBSP.md)):
> > http://www.domodom.fr/spip/Generer-une-image-Windows-CE-pour.html?lang=en

  * Customization of a WinCE image:
> > http://www.domodom.fr/spip/Customisation-de-notre-Image.html?lang=en

  * WinCE image upload:
> > http://www.domodom.fr/spip/Telecharger-une-image-Windows-CE.html?lang=en

  * A first application development:
> > http://www.domodom.fr/spip/Ma-premiere-application-sous.html?lang=en

  * WinCE drivers:
> > [GPIO](http://www.domodom.fr/spip/A-GPIO-driver-for-mini2440.html?lang=en),
> > [SPI](http://www.domodom.fr/spip/Un-driver-SPI-pour-la-mini2440.html?lang=en),
> > [I2C](http://www.domodom.fr/spip/Utiliser-le-driver-I2C-dans-une.html?lang=en),
> > [WiFi](http://code.google.com/p/friendlyarm/downloads/)

Note:
Some versions of the Supervivi bootloader are not compatible with WinCE 6.0 images.
It is possible that the bootloader of your board should be updated, see the following procedure: http://www.domodom.fr/spip/Mise-a-jour-du-bootloader.html?lang=en