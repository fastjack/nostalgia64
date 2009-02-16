; Die Routine läßt die X- und Y-Koordinaten unverändert. Ohne diese Beschränkung
; könnte man sich einige Kopieroperationen sparen.

	*=$c000
	!to "graph.prg", cbm
; Unbenutzte Stellen in der Zero-Page für Koordinaten verwenden
	x_coord = $03		; X-Koordinate in $03/$04
	y_coord = $05		; Y-Koordinate in $05
	
	bitmap_base = $2000	; Anfang des Grafikspeichers für Bitmaps
	scratch_space = $fb	; Puffer für Zwischenberechnungen (laut memory-map unbenutzt)
	operator1 = scratch_space
	operator2 = scratch_space+2
	offset = scratch_space	; hier liegt die adresse des bytes, in dem der punkt gezeichnet wird
	
jump_table:
	jmp graphics_on
	jmp graphics_off
	jmp clear_screen
	jmp plot
	
graphics_on:
	lda $d011
	ora #%00100000
	sta $d011	; hires-mode an
	lda $d018
	ora #%00001000
	sta $d018	; bitmap-speicher ab $2000
	rts
	
graphics_off:
	lda $d018
	and #%11110111
	sta $d018	; memory-bank auf normalwert setzen
	lda $d011
	and #%11011111
	sta $d011	; hires-mode aus
	rts
	
clear_screen:
	lda #<bitmap_base
	sta scratch_space
	lda #>bitmap_base
	sta scratch_space+1
	lda #0		; füllwert
	tay		; index
	tax		; zähler für 256-byte pages
clear_loop:
	sta (scratch_space),y
	iny
	bne clear_loop
	inc scratch_space+1	; nur hi-byte inkrementieren. rest wird über x-register addressiert
	inx
	cpx #$20		; 32*256 bytes gelöscht?
	bne clear_loop
	rts
	
; offset = (y_coord DIV 8) * 8 * 40 + (y_coord MOD 8) + (x_coord & %111)
; Uhhh, durch 8 und dann wieder mal 8? :)
; => y_coord & $fff8
plot:
	lda y_coord
	and #%11111000
	sta operator1
	lda #0		; Y-Koordinaten sind nur 1 byte lang
	sta operator1+1
	
	; * 40 (*8 + *32) ohne Schleifen, in der Hoffnung, daß es schneller ist
	asl operator1
	rol operator1+1
	asl operator1
	rol operator1+1
	asl operator1
	rol operator1+1	; mit 8 multipliziert
	
	lda operator1
	sta operator2
	lda operator1+1
	sta operator2+1	; kopieren

	asl operator2
	rol operator2+1
	asl operator2
	rol operator2+1	; und hier insgesamt mit 32 multipliziert
	
	; beides addieren und die untersten 3 bits der y-koordinate gleich dazurechnen. 
	clc
	lda operator1
	adc operator2
	sta operator1
	lda operator1+1
	adc operator2+1
	sta operator1+1
	clc
	lda y_coord
	and #%00000111
	adc operator1
	sta operator1
	lda operator1+1	; wäre eine combo aus bcc+inc vieleicht schneller?
	adc #0
	sta operator1+1
	
	; jetzt noch x_coord & %111 dazuaddieren
	lda x_coord
	and #%11111000
	sta operator2	; im lo-byte die untersten 3 bits löschen
	lda x_coord+1
	sta operator2+1	; hi-byte unverändert belassen

	clc
	lda operator1
	adc operator2
	sta offset
	lda operator1+1
	adc operator2+1
	sta offset+1
	
; Das Offset innerhalb des Grafikspeichers ist nun berechnet.

	clc			; jetzt noch die absolute adresse im speicher errechnen
	lda offset
	adc #<bitmap_base
	sta offset
	lda offset+1
	adc #>bitmap_base
	sta offset+1
	
	ldy #0
	lda x_coord
	and #%00000111		; nur die untersten 3 bits sind interessant
	tax
	lda (offset),y
	ora powers,x
	sta (offset),y
	rts
	
powers:	!byte $80, $40, $20, $10, $08, $04, $02, $01
