# GUI-BEATBOX-
Эта программа позволит писать простые музыкальные композиции  
(За основу взята программа BeatBox из книги Head First Java!!!)  
Графическое представление:  
![Menu](https://github.com/SssolidPrincesss/GUI-BEATBOX-/blob/main/GBeatBox/GB1.png)  
![Menu](https://github.com/SssolidPrincesss/GUI-BEATBOX-/blob/main/GBeatBox/GB2.png)   
![Menu](https://github.com/SssolidPrincesss/GUI-BEATBOX-/blob/main/GBeatBox/GB3.png)
  
Функционал:
1. Кнопки Start и Stop позволяют останавливать и возобновлять проигрывание музыкальной композиции
2. Tempo Up/Down отвечает за повышение/понижение темпа воспроизведения композиции
3. Счетчик БПМ показывает, с каким темпом воспроизводится композиция
4. Меню(File) дает возможность сохранить/загрузить файл и обновить поле (убрать флажки и скинуть темп до стандартного — 120 БПМ)
5. Светомузыка — фишка  
  
Теперь можно приступить к разбору кода программы  
    
Для начала необходимо добавить библиотеки для работы с MIDI, элементами GUI, вводом/выводом и коллекциями:
```java
import javax.sound.midi.*;
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.io.*;
import java.util.*;
```

Следующим шагом будет объявление переменных, заполнение строчного массива названиями инструментов, а также заполнение целочисленного массива значениями ключей для извлечения определенного звука из сиквенсора (javax.sound.midi):
```java
public class BeatBox {
    JPanel miniPanel;
    ArrayList<JCheckBox> checkboxList;
    Sequencer sequencer;
    Sequence sequence;
    Track track;
    JFrame theFrame;

    JCheckBox c;
    JLabel labelBPM;
    MyDrawPanel panel;
    int bpm = 120;
    
    String[] instrumntNames = {"Bass Drum", "Closed Hi-Hat", "Open Hi-Hat",
            "Acoustic Snare", "Crash Cymbal", "Hand Clap", "High Tom", "Hi Bongo", "Maracas", "Whistle", "Low Conga", "Cowbell",
            "Vibraslap", "Low-mid Tom", "High Agogo", "Open Hi Conga"};
    int[] instruments = {35,42,46,38,49,39,50,60,70,72,64,56,58,47,67,63};
```

В методе main мы просто создаем объект нашего класса и вызываем метод, который отвечает за создание GUI и пр.:
```java
    public static void main(String[] args) {
        new BeatBox().buildGui();
    }
```

Теперь о самом методе buildGui():
Эта маленькая фишка поможет программе создавать GUI в соответствии с настройками операционной системы:
```java
    public void buildGui(){
        try {
            UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
        } catch (ClassNotFoundException | InstantiationException | IllegalAccessException | UnsupportedLookAndFeelException e) {
            e.printStackTrace();
        }
```

В этом блоке кода мы создаем окно, устанавливаем операцию прекращения работы программы, создаем менеджер компоновки для панели заднего плана, устанавливаем границы для расположения элементов (окантовка) и красим панель:
```java
        theFrame = new JFrame("Cyber BeatBox");
        theFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        BorderLayout layout = new BorderLayout();
        JPanel background = new JPanel(layout);
        background.setBorder(BorderFactory.createEmptyBorder(10,10,10,10));
        background.setBackground(Color.PINK);
```



Здесь мы создаем лист для хранения чекбоксов, контейнер для хранения кнопок, а также сами кнопки (которым мы еще добавляем возможность взаимодействия):
```java
        checkboxList = new ArrayList<>();
        Box buttonBox = new Box(BoxLayout.Y_AXIS);

        JButton start = new JButton("Start");
        start.addActionListener(new MyStartListener());
        buttonBox.add(start);

        JButton stop = new JButton("Stop");
        stop.addActionListener(new MyStopListener());
        buttonBox.add(stop);
 
        JButton upTempo = new JButton("Tempo Up");
        upTempo.addActionListener(new MyUpTempoListener());
        buttonBox.add(upTempo);

        JButton downTempo = new JButton("Tempo Down");
        downTempo.addActionListener(new MyDownTempoListener());
        buttonBox.add(downTempo);
```

Тут мы создаем и настраиваем панель, на которую будет выводиться темп:
```java
        Font bpmFont = new Font("SansSerif", Font.BOLD, 12);
        labelBPM = new JLabel("Tempo: " + bpm + " BPM");
        labelBPM.setOpaque(true);
        labelBPM.setPreferredSize(new Dimension(100, 50));
        labelBPM.setBackground(Color.yellow);
        labelBPM.setFont(bpmFont);
        buttonBox.add(labelBPM);
```

Создаём контейнер для названий инструментов и заполняем его:
```java
        Box nameBox = new Box(BoxLayout.Y_AXIS);
        for (int i = 0; i < 16; i++) {
            nameBox.add(new Label(instrumntNames[i]));
        }
```

В этой части кода мы создаем отдельную панель для размещения кнопок и панели со светомузыкой, настраиваем ее видимость и правила компоновки, а потом добавляем в нее нужные элементы:
```java
        JPanel rightPanel = new JPanel();
        panel = new MyDrawPanel();
        rightPanel.setOpaque(false);
        rightPanel.setLayout(new BoxLayout(rightPanel, BoxLayout.Y_AXIS));
        rightPanel.add(panel);
        rightPanel.add(buttonBox);
```

Здесь мы настраиваем правила компоновки элементов на основной панели и добавляем их:
```java
        background.add(BorderLayout.EAST, rightPanel);
        background.add(BorderLayout.WEST, nameBox);

        theFrame.getContentPane().add(background);

        GridLayout grid = new GridLayout(16,16);
        grid.setVgap(1);
        grid.setHgap(2);
        
        miniPanel = new JPanel(grid);
        miniPanel.setBackground(Color.pink);
        background.add(BorderLayout.CENTER, miniPanel);
```
В этой части кода мы заполняем ранее созданную панель чекбоксами и заполняем ими же ранее созданную коллекцию, а также запускаем метод для настройки сиквенсора:
```java
        for (int i = 0; i < 256; i++) {
            c = new JCheckBox();
            c.setBackground(Color.ORANGE);
            c.setSelected(false);
            checkboxList.add(c);
            miniPanel.add(c);
        }
        setUpMidi();
```

В этой части кода создаем элементы меню и добавляем возможность взаимодействовать с ними, а также завершаем настройки фрейма:
```java
      JMenuBar menuBar = new JMenuBar();
    	JMenu fileMenu = new JMenu("File");
    	JMenuItem newMenuItem = new JMenuItem("New");
    	JMenuItem saveMenuItem = new JMenuItem("Save");
    	JMenuItem loadMenuItem = new JMenuItem("Load file");
    	newMenuItem.addActionListener(new NewMenuListener());
    	saveMenuItem.addActionListener(new SaveMenuListener());
    	loadMenuItem.addActionListener(new LoadMenuListener());
    	
    	fileMenu.add(newMenuItem);
    	fileMenu.add(saveMenuItem);
    	fileMenu.add(loadMenuItem);
    	menuBar.add(fileMenu);
    	
    	theFrame.setJMenuBar(menuBar);
      theFrame.setBounds(50,50,300,300);
      theFrame.pack();
      theFrame.setVisible(true);
    }
```

Далее идет метод для первичной настройки сиквенсора:
```java
    public void setUpMidi(){
        try{
            sequencer = MidiSystem.getSequencer();
            sequencer.open();
            sequence = new Sequence(Sequence.PPQ, 4);
            track = sequence.createTrack();
            sequencer.setTempoInBPM(120);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
```


Следующий метод позволяет проигрывать треки (при помощи вызова другого метода) и подстраивать под них светомузыку, а также производит необходимые настройки сиквенсора:
```java
    public void buildTrackAndStart(){
        int[] tracklist = null;
        sequence.deleteTrack(track);
        track = sequence.createTrack();
        int[] eventsIWant = {127};
        sequencer.addControllerEventListener(panel,eventsIWant);
        for (int i = 0; i < 16; i++) {
            tracklist = new int[16];
            int key = instruments[i];
            for (int j = 0; j < 16; j++) {
                JCheckBox jc = checkboxList.get(j+16*i);
                if(jc.isSelected()){
                    tracklist[j] = key;
                    track.add(makeEvent(176, 1, 127, 100, j));

                }
                else{
                    tracklist[j] = 0;
                }
            }
            makeTracks(tracklist);
            track.add(makeEvent(176,1,127,0,16));
        }
        track.add(makeEvent(192,9,1,0,15));
        try{
            sequencer.setSequence(sequence);
            sequencer.setLoopCount(sequencer.LOOP_CONTINUOUSLY);
            sequencer.start();
            sequencer.setTempoInBPM(120);
        }catch (Exception e){e.printStackTrace();}
    }
```


Метод, который отвечает непосредственно за воспроизведение звуков:
```java
    private void makeTracks(int[] list) {
    	
        for (int i = 0; i < 16; i++) {
            int key = list[i];
            if(key != 0){
                track.add(makeEvent(144,9,key,100,i));
                track.add(makeEvent(128,9,key,100,i+1));
            }
        }   
    }
```


Далее идут классы, которые позволяют настроить взаимодействие с элементами меню:
```java
    public class NewMenuListener implements ActionListener {   
    	public void actionPerformed(ActionEvent e) {
    		bpm = 120;
    		labelBPM.setText("Tempo: " + bpm + " BPM");
	        for (JCheckBox chB : checkboxList) {
	            chB.setSelected(false);
	        }
    	}
    }
    
  
    public class SaveMenuListener implements ActionListener {   
	    public void actionPerformed(ActionEvent e) {
	
	        boolean[] checkboxState = new boolean[256];
	        for (int i = 0; i < checkboxState.length; i++) {
	            if (checkboxList.get(i).isSelected()) {
	                checkboxState[i] = true;
	            }
	        }
	        JFileChooser fileSave = new JFileChooser();
	        fileSave.showSaveDialog(theFrame);
	        File selectedFile = fileSave.getSelectedFile();
	
	        if (selectedFile != null) {
	            saveFile(selectedFile, checkboxState);
	        }
	    }
	
	    private void saveFile(File file, boolean[] checkboxState) {
	        try (BufferedWriter writer = new BufferedWriter(new FileWriter(file))) {
	            for (boolean state : checkboxState) {
	                writer.write(state + "\n"); 
	            }
	        } catch (IOException ex) {
	            System.out.println("Ошибка при сохранении файла");
	            ex.printStackTrace();
	        }
	    
	    }
    }
   
    
    public class LoadMenuListener implements ActionListener{
    	
        public void actionPerformed(ActionEvent actionEvent) {
            JFileChooser fileOpen = new JFileChooser();
            fileOpen.showOpenDialog(theFrame);
            loadCheckboxState(fileOpen.getSelectedFile());
        }

        private void loadCheckboxState(File file) {
            boolean[] checkboxes = null;
            try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
                checkboxes = new boolean[256];
                for (int i = 0; i < 256; i++) {
                    String line = reader.readLine();
                    if (line != null) {
                        checkboxes[i] = Boolean.parseBoolean(line); 
                    } 
                }

                for (int i = 0; i < 256; i++) {
                    JCheckBox checkbox = checkboxList.get(i);
                    checkbox.setSelected(checkboxes[i]);
                }

                sequencer.stop();
                buildTrackAndStart(); 
            } catch (FileNotFoundException e) {
                System.out.println("Файл не найден: " + file.getName()); 
                e.printStackTrace();
            } catch (IOException e) {
                System.out.println("Ошибка при чтении файла: " + file.getName()); 
                e.printStackTrace();
            } catch (NumberFormatException e) {
                System.out.println("Ошибка: Неверный формат данных в файле.");
                e.printStackTrace();
            }
        }
    
    }
```
Важно!  
Файлы сохраняются с булевыми значениями для удобства загрузки. Сам файл выглядит так:    
![Menu](https://github.com/SssolidPrincesss/GUI-BEATBOX-/blob/main/GBeatBox/GB4.png)     
При сохранении(и загрузке) файла откроется дополнительный фрейм, в котором можно либо выбрать место для файла, либо открыть заранее сохраненный проект:  
![Menu](https://github.com/SssolidPrincesss/GUI-BEATBOX-/blob/main/GBeatBox/GB6.png)  
Внизу этого фрейма будет кнопочка «Сохранить» или «Загрузить» в зависимости от нашей цели:  
![Menu](https://github.com/SssolidPrincesss/GUI-BEATBOX-/blob/main/GBeatBox/GB7.png)  
![Menu](https://github.com/SssolidPrincesss/GUI-BEATBOX-/blob/main/GBeatBox/GB8.png)    
  
Очередной метод, связанный с сиквенсором, который создает MIDI-событие, заданное переданными параметрами:
```java
    public static MidiEvent makeEvent(int comd, int chan, int one, int two, int tick){
        MidiEvent event = null;
        try{
            ShortMessage a = new ShortMessage();
            a.setMessage(comd, chan, one, two);
            event = new MidiEvent(a, tick);
        }catch(Exception ex){

        }
        return event;
    }
```

В этой части кода мы настраиваем взаимодействие с кнопками:
```java
    public class MyStartListener implements ActionListener{
        
        public void actionPerformed(ActionEvent actionEvent) {
            buildTrackAndStart();
        }
    }
    public class MyStopListener implements ActionListener{
        
        public void actionPerformed(ActionEvent actionEvent) {
            sequencer.stop();
        }
    }
    public class MyUpTempoListener implements ActionListener{
        
        public void actionPerformed(ActionEvent actionEvent) {
            float tempoFactor = sequencer.getTempoFactor();
            sequencer.setTempoFactor((float)(tempoFactor * 1.03));
            bpm = (int) (bpm * 1.03);
            labelBPM.setText("Tempo: " + bpm + " BPM");
        }
    }
    public class MyDownTempoListener implements ActionListener{
        
        public void actionPerformed(ActionEvent actionEvent) {
            float tempoFactor = sequencer.getTempoFactor();
            sequencer.setTempoFactor((float)(tempoFactor * 0.97));
            bpm = (int) (bpm * 0.97);
            labelBPM.setText("Tempo: " + bpm + " BPM");
        }
    }
```

И, наконец, класс, который отвечает за прорисовку, обновление и другие действия, связанные со светомузыкой:
```java
    class MyDrawPanel extends JPanel implements ControllerEventListener{
		boolean msg = false;
		
		public void controlChange(ShortMessage event) {
			msg = true;
			repaint();
		}
		
		public void paintComponent(Graphics g) {
			if(msg) {
				Graphics2D g2 = (Graphics2D) g;
				
				int r = (int) (Math.random() * 250);
				int gr = (int) (Math.random() * 250);
				int b = (int) (Math.random() * 250);
				
				g.setColor(Color.black);
				g.fillRect (0, 0, this.getWidth(), this.getHeight());
				
				g.setColor(new Color(r, gr, b));
				
				int height = (int) ((Math.random() * 120) + 10);
				int width = (int) ((Math.random() * 120) + 10);
				int x = (int) ((Math.random() * 50) + 10);
				int y = (int) ((Math.random() * 100) + 10);
				
				g.fillRect(x, y, height, width);
				msg = false;
			}
		}	
	}  
}
```
