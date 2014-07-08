purePDF
=======

This is a porting of the java [iText] [1] library.
purepdf is a complete pdf library for actionscript which allows to read and create pdf document from any running swf files. It supports almost all the pdf features. For a complete view of the library features see the Examples page.


## Features

 * alpha gradient colors
 * support for pdf viewers display options
 * alpha transparency
 * layers and layers membership
 * support for pdf text rendering.
 * tables ( nested tables, page split tables, table with images, etc...)
 * slide show
 * page transitions
 * annotations
 * patterns, shadings patterns (linear and gradient), spot colors, rgb color and cmyk color
 * linear and radial gradients with alpha
 * forms (user input forms, textfields, combo box, list, checkbox).
 * paragraphs, phrases, chunks for text manipulation
 * image patterns
 * lists
 * basic and advanced paths
 * images ( png, tif, jpeg, bitmapdata, gif, animated gifs)
 * afm, otf, pfm, ttc and ttf fonts (embedded and not embedded)
 * metadata, page header and footers
 * external, internal links
 * barcodes creation ( ean-ucc 13, ucc-12, ean-ucc-8, upc-e, pdf 417, ean supplements)
 * unicode, cjk fonts
 * file attachments
 * arabic RTL writing
 * javascript
 * multi column text and text around shapes
 * page labels
 * vertical text
 * read existing pdf documents
 * and most of the pdf features..

See an auto-generated pdf with all the classes: http://www.sephiroth.it/purepdf/pdfs/Reflection.pdf


## Third party libraries:
  * [FZlib][2] 
  * [as3corelib][3]
  * [as3-commons][4]
  * Alchemy
  * [Actionscript HashMap][5]
  

## LICENSE

[The MIT License (MIT)][6]

[1]: http://itextpdf.com
[2]: http://www.wizhelp.com/fzlib/ 
[3]: http://code.google.com/p/as3corelib/
[4]: http://code.google.com/p/as3-commons/
[5]: http://code.google.com/p/ashashmap/
[6]: http://opensource.org/licenses/mit-license.php



html text to pdf with purepdf

05 Sep
Purepdf is a complete library for creating PDF documents using actionscript 3. This is ported from famous java iText library to actionscript 3.0. Previously I have got a chance to work with iText for a java project. This really helped me in working on purepdf for actionscript. Online API documentation for purepdf and example file doesn’t help much in building costume requirements.

My latest project got a requirement to convert html text to pdf document. I couldn’t find a direct way to convert in purepdf but I remember there is some alternate way in itext to do this, which is not available in purepdf. This motivated me in developing custom class that can create pdf text similar to html content in flash.

So here it goes..

import org.purepdf.Font;
import org.purepdf.colors.RGBColor;
import org.purepdf.elements.Chunk;
import org.purepdf.elements.Phrase;
import org.purepdf.pdf.PdfContentByte;
import org.purepdf.pdf.fonts.BaseFont;
import org.purepdf.pdf.fonts.FontsResourceFactory;
import org.purepdf.resources.BuiltinFonts;

/**
 * html text to purepdf phrase converter
 * @author sajeev
 */
class HTMLText
{
	private var _htmlText:String = "";
	private var phrase:Phrase;
	/**
	 * get the current text
	 * @return
	 */
	public function get text():String
	{
		return _htmlText;
	}
	/**
	 * set the new text
	 * @param value
	 */
	public function set text(value:String):void
	{
		phrase = null;
		_htmlText = value;
	}
	/**
	 * @param htmlText
	 */
	public function HTMLText(htmlText:String = null):void
	{
		FontsResourceFactory.getInstance().registerFont(BaseFont.TIMES_BOLD, new BuiltinFonts.TIMES_BOLD);
		FontsResourceFactory.getInstance().registerFont(BaseFont.TIMES_ROMAN, new BuiltinFonts.TIMES_ROMAN);
		FontsResourceFactory.getInstance().registerFont(BaseFont.TIMES_BOLDITALIC, new BuiltinFonts.TIMES_BOLDITALIC);
		FontsResourceFactory.getInstance().registerFont(BaseFont.TIMES_ITALIC, new BuiltinFonts.TIMES_ITALIC);
		this.text = htmlText;
	}
	/**
	 * returns current phrase text if present
	 * @return
	 */
	public function getPhraseText():Phrase
	{
		if(!phrase)createPhrase();
		return phrase
	}
	// build phrase obj
	private function createPhrase():void
	{
		if(phrase)return;
		phrase = new Phrase(null, null);
		//
		XML.ignoreWhitespace = false;
		htmlIterator(new XML("<data>" + inserCDATA(_htmlText) + "</data>"));
	}
	// insert cdata on required areas
	private function inserCDATA(xml:String):String
	{
		var replaceFunction:Function =  function():String{
			var val:String = arguments[1];
			if(val && val.length)
				return "><![CDATA["+ val +"]]><";
			else
				return arguments[0];
		}
		return xml.replace(/>([^<]*?)</gi, replaceFunction);
	}
	// loop that creates info to create phrase from given html text
	private function htmlIterator(content:XML, info:Array = null):void
	{
		info = info ? info.slice(0) : [];
		if(content.name())
			info.push({name:content.name().localName, value:content.toXMLString().replace(/^[^<]*(<[^>]*?>)[sS]*$/,"$1")});
		if(content.hasComplexContent())
			for each(var item:XML in content.children())
			htmlIterator(item, info);
		else
			createChunk(content.toString(), info);
	}
	//crease each individual chunk and add that to phrase on finding each text piece
	private function createChunk(content:String, info:Array):void
	{
		content = replaceHtmlChracter(content);
		//
		var dataDetail:String;
		var detaiXml:XML;
		//
		var fontSize:Number = 12;
		var fontColor:uint = 0;
		var fontName:String;
		var fontBold:Boolean = false;
		var fontItalic:Boolean = false;
		var fontUnderline:Boolean = false;
		//
		for each(var data:Object in info){
			switch (data.name.toUpperCase()){
				case "FONT":
					dataDetail = data.value;
					detaiXml = new XML(dataDetail.replace(/^(.*?)/{0,1}>$/,"$1/>"));
					if(detaiXml.@SIZE.length())
						fontSize = parseInt(detaiXml.@SIZE.toString());
					if(detaiXml.@COLOR.length())
						fontColor = parseInt(detaiXml.@COLOR.toString().replace(/#/,""), 16);
					//TODO
					if(detaiXml.@FACE.length())
						fontName = detaiXml.@FACE.toString();
					break;
				case "U":
					fontUnderline = true;
					break;
				case "B":
					fontBold = true;
					break;
				case "I":
					fontItalic = true;
					break;
				default:
					break;
			}
		}
		//
		var font: Font;
		if(fontBold && fontItalic)
			font = new Font(Font.TIMES_ROMAN, fontSize, Font.BOLDITALIC , RGBColor.fromARGB(fontColor));
		else if(fontBold)
			font = new Font(Font.TIMES_ROMAN, fontSize, Font.BOLD , RGBColor.fromARGB(fontColor));
		else if(fontItalic)
			font = new Font(Font.TIMES_ROMAN, fontSize, Font.ITALIC , RGBColor.fromARGB(fontColor));
		else
			font = new Font(Font.TIMES_ROMAN, fontSize, Font.NORMAL , RGBColor.fromARGB(fontColor));
		//
		var chunk: Chunk = new Chunk(content, font);
		if(fontUnderline)
			chunk.setUnderline(RGBColor.fromARGB(fontColor), 1, 0, -2, 0, PdfContentByte.LINE_CAP_ROUND);
		//
		phrase.add(chunk);
	}
	// reaplace html special character with original one
	private static function replaceHtmlChracter(text:String):String
	{
		return text.replace(/&lt;/ig,"<")
			.replace(/&gt;/ig,">")
			.replace(/&amp;/ig,"&")
			.replace(/&quot;/ig,'"')
			.replace(/&copy;/ig,"©")
			.replace(/&reg;/ig,"®")
			.replace(/&ndash;/ig,"–")
			.replace(/&mdash;/ig,"—")
			.replace(/&nbsp;/ig," ")
			.replace(/&frasl;/ig,"/");
	}

}
Below are the list of html tags that are already implemented :-

fontsize
fontcolor
bold
italic
underline
Now how to use this!

var writer: PdfWriter = PdfWriter.create( new ByteArray(), null);
var document: PdfDocument = writer.pdfDocument;
document.open();
var htmlText:HTMLText = new HTMLText();
htmlText.text  = "<b>my html text</b>"
var phrase: Phrase = htmlText.getPhraseText();
document.add(phrase);
document.close();
