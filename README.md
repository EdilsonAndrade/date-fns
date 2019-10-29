# date-fns é uma biblioteca javascript interessante para se trabalhar com datas
### Instalar utilizando este comando abaixo que fará a instalação da ultima versão

<pre>
yarn add date-fns@next 
</pre>

## Alguns dos métodos interessantes e mais utilizados são conforme a importação abaixo:
<pre>
import { startOfHour , parseIso, isBefore, isAfter, format, setMinutes, setSeconds, setHour } from 'date-fns'
</pre>

# Codigo de exemplo e utilização dos metodos importados
Nesta classe simples é realizado a busca no banco de dados que carrega a agenda de um determinado prestador de serviço onde é demonstrado ao usuário os horários que o mesmo tem disponivel dentro de uma janela de atendimento do mesmo e se não tiver ocupado.

```javascript
import {
  startOfDay,
  endOfDay,
  isAfter,
  setHours,
  setMinutes,
  setSeconds,
  format,
} from 'date-fns';
import { Op } from 'sequelize';
import Appointements from '../models/Appointment';

class AvailableController {
  async index(req, res) {
    const { date } = req.query;

    if (!date) {
      return res.status(400).json({ error: 'Invalid date' });
    }

    const searchDate = Number(date);

    const appointements = await Appointements.findAll({
      where: {
        provider_id: req.params.providerId,
        canceled_at: null,
        date: {
          [Op.between]: [startOfDay(searchDate), endOfDay(searchDate)],
        },
      },
    });
    
    // janela de atendimento de um prestador de serviço, melhor ficar em banco, mas foi para efeito de pratica.
    const schedule = [
      '08:00',
      '09:00',
      '10:00',
      '11:00',
      '12:00',
      '13:00',
      '14:00',
      '15:00',
      '16:00',
      '17:00',
      '18:00',
      '19:00',
      '20:00',
      '21:00',
      '22:00',
    ];

    const available = schedule.map(time => {
      const [hour, minute] = time.split(':');
      const value = setSeconds(
        setMinutes(setHours(searchDate, hour), minute),
        0
      );
      return {
        time,
        value: format(value, "yyyy-MM-dd'T'HH:mm:ssxxx"),
        available:
          isAfter(value, new Date()) &&
          !appointements.find(a => format(a.date, 'HH:mm') === time),
      };
    });

    return res.json(available);
  }
}

export default new AvailableController();

```
