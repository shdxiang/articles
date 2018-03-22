.PRECIOUS: %.pdf
.PHONY: clean

all: \
	bitcoin-zh-cn-1.svg \
	bitcoin-zh-cn-2.svg \
	bitcoin-zh-cn-3.svg \
	bitcoin-zh-cn-4.svg \
	bitcoin-zh-cn-6.svg \
	bitcoin-zh-cn-6.svg \
	bitcoin-zh-cn-7.svg \
	bitcoin-zh-cn-8.svg \
	bitcoin-zh-cn-9.svg \
	bitcoin-zh-cn-10.svg \
	bitcoin-zh-cn-11.svg \

%.svg: %.pdf
	pdf2svg $< $@

%.pdf: %.tex
	xelatex $<
	pdfcrop $@ $@.crop
	mv $@.crop $@

clean:
		rm -f *.log *.aux
