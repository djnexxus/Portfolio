# Portfolio
This is my portfolio of some of my work. Snippets of code that was produced in the actual production environment and a breif explanation of its application.

Project : Hanheld Barcode Scanner
Issue : Client aquired this scanner and was unable to integrate it into their existing production system due to a combination of software incompatabilities and cost of ownership. The issue was resolved using a custom php SAS which was accessed by the scanner through it's default web browser and allowed the system to scan directly to the pre-existing database. 

<img src="php/handheld.jpg" class="center" alt="Barcode Scanner" title="Barcode Scanner">

```php
public function scan(Request $request)
    {
        // Validate the incoming data and ensure the "Serial Number" passes the regex test.
        $this->validate($request, [
            'mode'=>'required',
            'skid'=>'required',
            'salesorder'=>'required',
            'quantity'=>'required',
            'serial'=>'required|regex:/^\d{3}-\d{4}/'
        ]);

        // If the data contains 14 digits, classify it as a serial number and create a database record.
        if(Str::length($request->serial) == 14){
            $record = Shipment::where('serial', '=', $request->serial)->first();
            if($record === null){
                Shipment::create([
                    'mode'=>$request->mode,
                    'skid'=>$request->skid,
                    'salesorder'=>$request->salesorder,
                    'quantity'=>'1',
                    'product'=>Str::substr($request->serial, 0,8),
                    'serial'=>Str::substr($request->serial, 9,5),
                ]);
                $errMessage = 'Valid Serial';
            }else{
                $errMessage = 'Duplicate Serial';
            }

        // If the data is only 8 characters in length classify it as a part and create the appropriate record.
        }elseif(Str::length($request->serial) == 8){
            Shipment::create([
                'mode'=>$request->mode,
                'skid'=>$request->skid,
                'salesorder'=>$request->salesorder,
                'quantity'=>$request->quantity,
                'product'=>Str::substr($request->serial, 0,8),
            ]);
            $errMessage = 'Valid Part';
        }else{

        }
        // Return the error message to the view to update the user of the status of the scan.
        return view('scanner', ['errMessage'=>$errMessage]);
    }
```
